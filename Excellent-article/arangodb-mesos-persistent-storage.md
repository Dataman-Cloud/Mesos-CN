## 如何利用Mesos持久化存储方案部署ArangoDB 集群

调度分配无状态应用的 Docker 容器很方便，但是该如何处理那些需要保持状态的应用呢？

我们无法轻易的停止一个挂载了本地文件系统的 Docker 容器并在另一台机器上重启它。然而，出于性能的考量，分布式数据库和其他有状态的服务通常又需要使用本地存储，甚至是 SSD 盘。所以数据中心操作系统需要考虑为应用提供保留并访问本地存储的方式；确保某些任务重启后被重新调度到同一个节点，进而重新使用它的本地存储。

[Mesos 0.23](http://www.mesoscn.cn/Release/mesos-023-released.html) 的 persistence primitives 特性解决了上述这些挑战。本文中，我们首先解释下 [Mesos](http://www.mesoscn.cn/) 的 persistence primitives 特性是如何工作的，其次会给出一个具体的例子，一个 Mesosphere 认证的使用 persistence primitives 特性的框架： ArangoDB —— 分布式 NoSQL 数据库。

### 概述

为了确保任务失败后仍然能够访问本地持久化的数据，Mesos需要解决下面两个问题：

1. 为了在同一个台节点上重启任务，框架需要能够再次收到该节点的资源offer。

2. 在默认的情况下，任务停止后，Mesos会清除掉该任务沙盒里面的数据，但是对于需要持久化的数据，该如何避免任务失败或重启后数据被清除掉呢？

通过配合使用不同的 persistence primitives，我们就可以解决以上问题。

### 动态保留

因为 Mesos 的动态保留特性允许框架保留曾提供给它的资源，并且被保留的资源不会提供给其它 role，从而框架可以借助该特性来将任务再次部署到同一个节点上，该节点上已经存储了任务需要的数据。关于 Mesos 动态保留技术的细节，请参考链接：[https://github.com/apache/mesos/blob/master/docs/reservation.md](https://github.com/apache/mesos/blob/master/docs/reservation.md)

### 持久卷

框架利用 Mesos 的特性可以在磁盘资源上创建持久化卷。持久化卷是任务沙盒之外的资源，不会因任务异常退出或完成而被清除。任务退出时，任务的资源（包括持久化卷）会被重新提供给该任务所属的 role，从而该 role 下面的框架可以在相同的节点上启动原来的任务，或者启动修复任务，或者启动新的任务来消费前一个任务的输出（这些输出已经被存储到了持久化卷中）。另外值得注意的是，我们只能从动态保留的磁盘资源上创建持久化卷。

关于如何创建和销毁持久化卷的技术细节，请参考链接：[https://github.com/apache/mesos/blob/master/docs/persistent-volume.md](https://github.com/apache/mesos/blob/master/docs/persistent-volume.md) 另外，Mesos也给出了操作持久化卷的例子：[https://github.com/apache/mesos/blob/master/src/examples/persistent_volume_framework.cpp](https://github.com/apache/mesos/blob/master/src/examples/persistent_volume_framework.cpp)


### 使用 persistence primitives 的最佳实践

下面是 persistence primitives 的一些入门技巧：

* 在一个 acceptOffers 调用里面同时创建动态保留和持久化卷
* 由于网络异常，或者 master 拒绝操作等，框架在尝试动态保留资源或创建持久化卷时可能会失败，这时框架需要检查这些失败并处理失败的情况（譬如 重试机制）。
* 通过mesos调度的API很难监听失败的动态保留情况，因为保留的资源没有唯一的ID，而且，对于一个动态保留请求成功与否，Mesos master没有提供明确的反馈。在下面的ArangoDB的例子中，给出了处理这种情况的技巧。另外，Mesos 未来会支持为资源保留打标签的功能，[https://docs.google.com/document/d/1HdS4TxmfMkMR_2mvOHZR-LLmyLNdyqRrPBdiGCp6-w8/edit](https://docs.google.com/document/d/1HdS4TxmfMkMR_2mvOHZR-LLmyLNdyqRrPBdiGCp6-w8/edit) 这会简化上述流程。
*  因为有许多需要处理的“状态”，我们最好为持久化框架的应用逻辑维护一个状态机，处理应用从初始状态（没有保留资源和持久化卷），到某个结束状态（需要的资源得到了保留，并且持久化卷创建成功）的过渡。下面 ArangoDB 的例子给出了更多细节。
* 由于持久化卷与 role 有关，一个卷可能会被提供给该 role 下的任何一个框架。譬如，一个持久化卷可能是由框架 A 创建的，然后被 offer 给了同一个 role 下的框架 B。利用这一点，可以很容易的在框架之间分享该卷下的数据。然而，从另一方面看，这也可能导致框架 A 的敏感数据被框架 B 获取到。

关于在 持久化卷特性上编程的更多细节，请参考链接：[https://github.com/apache/mesos/blob/master/docs/persistent-volume.md#programming-with-persistent-volumes](https://github.com/apache/mesos/blob/master/docs/persistent-volume.md#programming-with-persistent-volumes)

### ArangoDB 是什么？以及它为什么需要持久化？

ArangoDB 是一个分布式的多模型数据库，它是一个包含文档存储、key-value 存储和图存储的数据库，所有的操作都基于同一个引擎，使用统一的 query 语法。

作为一个分布式应用，由于 Apache Mesos 或者[数据中心操作系统（DCOS）](https://www.shurenyun.com/)负责分发和调度应用，管理集群资源和失败检查，ArangoDB 运行在其上会有很多好处。Mesos 也可以利用 Docker 容器部署应用，从而为开发者解决了很多分布式系统下令人头疼的问题。

另外，对数据库来说，一个非常大的需求是维护不停变化的状态，同时由于数据库需要数据持久化，ArangoDB 需要访问持久化的数据。理想情况下，甚至需要使用本地的 SSD 数据盘（现在，这种 SSD 盘能够提供非常好的IO性能，数据持久化保障以及智能预测的数据缓存能力。）

与利用分布式文件系统托管上面承载的分布式应用的方法不同，针对于ArangoDB，我们决定使用下面的方法来进一步提高数据持久化性能，即，所有的 ArangDB 集群的任务都访问本地存储，并在重启（譬如版本升级导致的重启）后能够仍然访问他们的数据。这就是我们选择利用 Apache Mesos 的 persistence primitives 特性的原因。

### 概要及不同的任务类型

有四种不同的 ArangoDB 任务：

* **Agency 任务**：Agency 是一个中心化，低数据量，低IO的数据存储，主要存储了 ArangoDB 集群的配置以及数据库/集合的摘要信息。 Agency 也使用了 Raft 一致性协议来维护集群层面的日志同步操作和锁操作。Agency 的数据变化不频繁，但是对整个集群的运转特别重要。这也意味着 Agency 的数据需要使用持久化卷，但是 IO 性能并不重要。
* **Coordinator 任务**：Coordinator 任务就是面向客户端的 ArangoDB 实例。它们本质上是无状态的：它们接受外部的 HTTP 请求，通过 Agency 了解集群的配置，并将 query 操作分发到集群中。Coordinator 知道集群切片的细节，从而能够制定分布式的query执行策略，并管理它们的运行状况。
* **主数据服务任务**：主数据服务器保存了真正的数据。这些任务状态频繁变化，有很重的数据IO操作。从而，这些任务需要持久化卷和很好的磁盘IO性能。
* **备数据服务任务**： 备数据服务器是主数据服务器的异步分片。它们有规律的轮训主数据服务器的变化并将这些变化应用到它们的数据备份上，所以它们也需要持久化卷。另外，如果主数据服务器崩溃了，它的备数据服务器能够立刻接管。

### ArangoDB 调度器的职责

ArangoDB集群按下面的顺序加载：

1. 加载 Agency 任务。
2. 主数据服务器启动。
3. 备数据服务器在主数据服务器之后启动，确保其不与主数据服务器运行在同一个节点上。
4. Coordinator 任务启动。
5. 在所有的任务启动并运行正常后，ArangoDB 集群执行初始化。

在下面的情况下，ArangoDB 调度器也需要采取行动并维护类似的依赖需求：

* 集群需要在数据服务器层面伸缩来改变它的数据存储能力。
* 集群需要在 Coordinator 层面伸缩来改变它的 query 能力。
* 集群需要不停服升级。
* 集群需要重启。

ArangoDB 调度器也需要进行失败处理。接到 Mesos master 的任务被杀或者 Mesos 节点失联的通知后，调度器需要自动的进行失败处理。为了进行失败处理，调度器会监听集群中任务的状态，维护时间戳，并进行超时处理。

ArangoDB 通过一个事件轮训的主程序来完成上述功能。这个主程序周期性从其他线程获取资源 offer，接受 Mesos 的更新信息，维护超时处理和事件响应。我们会在下面详细讨论这里的技术细节。

### ArangoDB 服务器的状态图

下面是使用了持久化卷的ArangoDB集群的一个任务状态图。（以一个数据服务任务为例）

{<1>}![](/content/images/2016/02/20160225StateDiagramm-600x450.png)

##### “NEW” 状态

任务从 “NEW” 状态开始：此时任务还没有动态保留资源和持久化卷。调度器收到了满足条件的资源offer后会返回一个 类型为 “RESERVE” 的 “mesos::Offer:Operation” 信息来尝试为 role —— arangodb 保留足够的资源。接着，任务状态变为 “TRY TO RESERVE” 。

##### “TRY TO RESERVE” 状态

Mesos master 通常会从同一个节点向上述任务再发送一次资源 offer，这次的 offer 中将包含为 role —— arangodb 动态保留资源的标示。此后，调度器会返回一个类型为 “CREATE” 的 “mesos::Offer::Operation” 信息来尝试创建持久化卷。接着任务状态变为 “TRY TO PERSIST”。持久化卷拥有一个全局唯一的 ID 标识。

##### “TRY TO PERSIST” 状态

Mesos master 从同一个节点再发送一次资源 offer 来响应上述的创建请求，这次的 offer 中将包含动态保留资源的标示和持久化卷。最终，Docker 容器启动并挂载持久化卷到容器上。任务状态变为 “TRY TO START"。

##### “TRY TO START” 状态

任务初始化完成后，Mesos Master 通知调度器任务启动成功。调度器将状态置为 “RUNNING”。

在前面的任务变为 “TRY TO START” 状态后，调度器开始启动依赖于这些任务的其它任务。若集群中所有的任务启动成功，调度器开始进行集群的全局初始化流程。

若在某些节点上有为role —— arangodb 静态保留的资源，调度器能够直接从状态 NEW 变为状态 “TRY TO PERSIST”。在这种情况下，调度器能够立刻获取到满足需求的资源 offer 并创建持久化卷。

由于存在消息丢失的情况，从 “TRY TO RESERVE” 到 “TRY TO START” 所有的 “TRY” 状态都有超时处理。如果状态超时，调度器会把状态重置为 NEW 来从节点接收新的资源 offer。

##### “RUNNING” 状态和 “FAILURE” 状态

理论上来说，“RUNNING” 状态的任务不需要跟调度器交互。但是，由于任务可能死掉，甚至会出现机器硬件失效导致 Mesos 节点整个失联，或者，计划内的停机维护，或者网络异常等，这些都会导致任务中断。

一个容错的分布式系统必须优雅的处理上述情况。持久化卷支持任务快速重启，并重连到集群中继续工作。调度器收到任务被杀（或者节点失联）的消息后，会把状态变为 KILLED 并等待资源 offer。

假设网络是正常的，任务所在节点的资源 offer 将如常到达调度器。这个 offer 包含了任务的保留资源和标记了全局唯一 ID 的持久化卷。调度器接收到 offer 后将立刻尝试重启任务并把状态变为 “TRY TO RESTART”。除了超时处理不同， “TRY TO RESTART” 状态本质上与 “TRY TO START” 状态等价。在 “TRY TO RESTART” 状态下，当响应超时时，任务状态变回 “KILLED”，而不是 “NEW”。 对于失败处理，调度器有另一个更长的超时处理机制。

如果被杀掉的主数据服务没能够尽快重启，调度器会把主数据服务切换为备数据服务，同时原来的备数据服务立刻变为主数据服务对外提供服务。原来的主数据服务状态变为 “FAILED OVER”。 原节点上存在的持久化数据可以使得任务能够更快恢复，所以调度器仍然会尝试在原节点上重启任务。

最终，在更长的时间后，若任务仍然重启失败，调度器将放弃这个任务和它的持久化数据并把状态从 “FAILED OVER” 变回 “NEW”。 在这种情况下，调度器会认为任务死亡并创建新的任务。接着调度器会销毁死亡任务的持久化卷并释放动态保留的资源。

若任务死亡后，调度器没有清除数据， Mesos 集群会发生资源泄漏：这些资源将被我们的框架永久占有而无法为其他框架所用。另外，由于该卷的唯一 ID 永久绑定在了特定的任务实例上，我们的框架也无法再度使用这个持久化卷了。

为了解决上述问题，调度器收到带有持久化卷或者动态保留的资源 offer，而该 offer 又没有对应的等待任务时，它将回复 “mesos::Offer::Operation”去销毁持久化卷或者释放相应的动态保留资源。这也可以归还创建超时时的遗留资源。

### 目标，计划和当前

与其它分布式系统一样，调度器本身也需要容错。我们通过下面两步来实现。首先，通过 Marathon 部署调度器实例。其次，利用 Apache Mesos 的状态模块来持久化调度器的内部状态，本质上是将数据存储到了 Zookeeper 上。这样在调度器新的实例启动成功后，它能够从 Mesos 获取原来实例的状态并正常运行。

为了保证无缝工作，我们实现了调和协议：若调度器已经获取了原来的状态，它首先会尝试与 Mesos 提供的正在运行任务的状态进行调和。即，调度器会将相应的已知任务信息发送给Mesos master，Mesos master 将这些信息更新后返回给调度器。

调度器持久化状态分为三个部分：目标，计划和当前。 ArangoDB 集群用户可以设置集群的“目标”，譬如数据服务实例的数量，coordinator 的实例数量等。调度器会一直监控集群“目标”，并通过修改“计划”来响应其变化。

“计划”比“目标”更具体，譬如它包含了各个任务的状态，以及调度器想要每个任务达到的状态等。由于“计划”是机器生成的信息，它将会一直保持在一个明智的状态下，从而会给出一个切实可达的状况。

调度器会根据Mesos发来的信息频繁更新 “当前”状态。“当前”状态反应了ArangoDB 集群的当前情况，譬如成功启动的任务，杀掉的任务等。调度器会通过操作相应的资源 offer 来持续尝试将“当前”状态变为“计划”状态。

“目标”，“计划”与“当前”三态是搭建一个容错，自愈的分布式系统常用的方法。希望它在更多的类似系统中发挥作用。考虑到稳定与自愈的特性，我们在 ArangoDB 集群中使用同样的方法得到了非常好的效果。

### 结论

Mesos 的 persistence primitives 是一个新的强大的工具，它使得更多的有状态应用可以运行在 Mesos 上。利用它，我们把 ArangoDB 集群改造成了一个极易扩展到数百节点的分布式持久化框架。

我们在这里分享了我们开发该系统的经验。考虑到大量的不同状态和超时情况，我们建议把持久化卷的逻辑模型化为状态机。这种方法也能够帮助你在调度器中实现很重要的失败处理，从而构建一个稳定的有状态框架。

>转自[数人云](https://www.shurenyun.com/)博客：[如何利用Mesos持久化存储方案部署ArangoDB 集群](http://blog.dataman-inc.com/20160225-arangodb-mesos-persistent-storage/)
