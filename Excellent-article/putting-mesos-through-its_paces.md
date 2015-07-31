#Autodesk如何使用Mesos来实现自己的PaaS云平台
Autodesk, 是一家收入超过25亿美元的跨国公司，拥有全球领先的3D设计、工程和娱乐软件，帮助客户构思、设计并创建一个更美好的世界。

Autodesk 希望在第三个十年持续增长，借助 Apache Mesos 和 Mesosphere 增强现有的基础设施。其主要的目标是加快其 bleeding-edge cloud 的更新速度，这个产品主要应用于仿真、分析、模拟和聚合。

在Autodesk眼里，以 Mesos 为核心的平台，可以对底层复杂基础设施进行抽象。  Autodesk 云服务采用了混合云部署，分别在 AWS 和 私有数据中心中，但是这些细节不需要用户去关心。

"我们的云部署在哪里，这不是开发者需要关心的事情"，云服务的技术总监 Stephen Voorhees 说到。“我们的目标是让用户想部署应用的时候，告诉云服务他需要哪些资源就好。不再需要自己来部署虚拟机，或者调用云平台基础资源有关的各种 API”

我们之所希望使用 Mesos 技术， 因为它可以将 Autodesk 的基础设施和数据中心进行抽象。Mesos 不但能够高效率地分配底层资源，而且能够有效地提高资源利用率。

Vorhees 说：“我们真的不想处理去了解哪些型号的服务器在哪个数据中小运行这些细节。我们仅仅想按一个按钮，然后通过持续集成系统将容器部署到 Mesos 中。”

虽然越来越多的同事认识到 Mesos 能够提供我们需要的资源抽象能力，公司还是需要通过测试来证明可行性。

##Putting Mesos through its paces

Olivier Paugam，Autodesk 云平台高级软件架构师和主力码农，开始接手这个 POC 项目。他想看看 Mesos 的能力是否货真价实，所以建立了一个以 Mesos 为基础的环境来部署公司新开发的 Event Streaming Service。目的是证明 Mesos 为核心的环境能够在跨组织、跨部门甚至跨公司之间运作。（Paugam 介绍了自己关于 Mesos 的经验 [Autodesk Cloud Engineering blog](http://cloudengineering.autodesk.com/blog/)， 这些帖子是非常值得一读）。

在 Paugam 看来，Event Streaming Service 已经成为我们云平台核心组件中的关键部分。它处理大量后端系统之间的通信，包括图形渲染、数据转换、分析和身份管理。你可以相像一下该服务需要处理的数据的规模和数量 —— 这些数据被不同的组件发布和订阅，整个过程还要保证高度的可靠性。
 
 
Event Streaming Service 已经投入了生产环境，到目前为止测试的结果给人留下了深刻的印象。该系统部署在 AWS 全球四个独立的区域，每个集群大约有12台服务器。基础平台基于 Mesos ， Mesos， [Marathon](https://mesosphere.github.io/marathon/) 和 [Ochopod](https://github.com/autodesk-cloud/ochopod), 包括  ZooKeeper， Kafka， RabbitMQ， Redis，HAProxy 和一些 Play framework web staks.

![](https://mesosphere.com/wp-content/uploads/2015/07/ochothon-800x505.png)

Ochopod 是 Autodesk 基于 Python 开发的开源项目，可以将容器组织起来一同工作（orchestration），并且能够与 Kubernetes 或者 mesos 进行整合。Autodesk 还开源了另外一个项目 Ochoton，作为组件它能轻易地整合到 Mesos/Marathon 里面，用来管理和部署 Ochopod 容器，实现了一个简化的 PaaS。


当 Event Streaming Service 投入到生产中后，Autodesk 预计每个集群规模会增长到 30 台服务器。“在 Mesos 的帮助下，我们能够轻易地扩大集群规模，应对不断增长的压力”

Paugam 也和 Voorhees 一样, 希望开发人员和底层资源之间的抽象层程度越高越好。Mesos + Docker 的解决方案果然每月让大家失望。

Paugam 兴致勃勃地描述道：”当我们将 Mesos 作为通用的资源平台，结果发生了天翻地覆的变化。这实在真是太棒了！  Mesos 提供的强大的解耦功能正是我们想要的，这是一个巨大的进步。“

Paugam 回忆以前要部署一个类似的服务需要花费数周的时间。首先要想运维部门申请新的 Amazon EC2 镜像，当镜像完成之后，再找出如何部署代码并且连接到 Kafka，ZooKeeper 和其他组件。如果机器宕机或者他们的配置发生变化，这通常意味着还要重新部署代码。


使用 Mesos 之后，部署应用只需要三个步骤，操作一组镜像，确保镜像正常建立，然后部署自己的容器。

![](https://mesosphere.com/wp-content/uploads/2015/07/eventstreaming-800x618.jpg)


##让 Mesos 在 Autodesk 里面唱上主角


Event Streaming Service 项目给 Autodesk 的同事留下了深刻的印象，大家相信 Mesosphere 的商业化的数据中心操作系统（Datacenter Operating System ——— DCOS）是落地 Mesos+Marathon 的好方法。虽然最初的出发点是简化开发者部署应用的难度，后来 Autodesk 也希望能够很容易的管理并且更新 Mesos 系统本身。


在复杂的部署应用程序中， 开发商倾向于  Autodest  希望消除设置和拓展  Mesos  环境的业务复杂性并保持所有组件的最新。

Vorhees说：“我们也不想关心 Mesos 部署和管理的细节，我们只希望知道 —— Mesos就在那里，干它应该干的事情”。简单、高效和实用是 Autodesk 中广泛采用 Mesos 的原因。其他各种各样的大数据框架，包括 Hadoop 和 Spark 也能在 Mesos框架上运行。


最终，提供一个统一的、可伸缩的内部平台，用于管理内部各种各样的系统将是一个巨大的胜利。“Autodesk 经历了多年终于找到一个部署我们软件的标准方法”，Vorhees 说到。“我们认为 Mesos，marathon 和 Docker 的组合解决了过去遇到的很多问题。”

因为有了 Mesos 提供的标准化的基础设施和应用部署方法，Autodesk 现在开始下一个大的跃进 —— 想第三方开发者提供一个 PaaS，这将是 Mesos 在 Autodesk  当仁不让的任务。







