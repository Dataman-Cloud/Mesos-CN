##Mesos 高可用性

&nbsp;&nbsp;&nbsp;&nbsp;Mesos 利用多台 Mesos master 来实现高可用性（high-availability），包括一个活跃的 master （叫做 leader 或者 leading master）和若干备份 master 来避免宕机。 通过 Apache ZooKeeper 选举出活跃的 leader，然后通知集群中的其他节点，包括其他 Master，slave节点和调度器（scheduler driver）。

关于 Apache ZooKeeper 原理，可以在[ Apache ZooKeeper ](http://zookeeper.apache.org/doc/trunk/recipes.html#sc_leaderElection)官网中找到。本文假定您已经知道如何运行并配置 Apache ZooKeeper，它的客户端类库已经包含在标准 Mesos 发行版中。

###用法
设置 Mesos 为高可用模式：

1. 确保 ZooKeeper 集群已经正常启动和运行。

2. 提供所有的 master，slave和 framework scheduler (调度框架) 的 znode 路径。如：

    - 使用 ***--zk*** 标志启动 mesos master二进制文件，例如 ***–zk=zk://host1:port1,host2:port2,…/path'***。

    - 启动 mesos-slave 执行文件，通过 ***--master=zk://host1:port1,host2:port2,.../path***  。

    - 使用以上的 ***zk*** 路径启动各种 framework scheduler。请注意 SchedulerDriver 必须使用这个路径来创建。

从现在开始 Mesos master 和 slaves 都与 ZooKeeper 进行交互，找出当前处于 leader 状态的 master。 

###实现细节

Mesos implements two levels of ZooKeeper leader election abstractions, one in src/zookeeper and the other in src/master (look for contender|detector.hpp|cpp).

The lower level LeaderContender and LeaderDetector implement a generic ZooKeeper election algorithm loosely modeled after this recipe (sans herd effect handling due to the master group’s small size, which is often 3).

The higher level MasterContender and MasterDetector wrap around ZooKeeper’s contender and detector abstractions as adapters to provide/interpret the ZooKeeper data.

Each Mesos master simultaneously uses both a contender and a detector to try to elect themselves and detect who the current leader is. A separate detector is necessary because each master’s WebUI redirects browser traffic to the current leader when that master is not elected. Other Mesos components (i.e. slaves and scheduler drivers) use the detector to find the current leader and connect to it.

The notion of the group of leader candidates is implemented in Group. This abstraction handles reliable (through queues and retries of retryable errors under the covers) ZooKeeper group membership registration, cancellation, and monitoring. It watches for several ZooKeeper session events:

Connection
Reconnection
Session Expiration
ZNode creation, deletion, updates

通过 ***MASTER_CONTENDER_ZK_SESSION_TIMEOUT*** 和  ***MASTER_DETECTOR_ZK_SESSION_TIMEOUT*** 我们明确定义了 session 的 timeout 时长。当 Master 节点和 Zookeeper 联络中断超过了这个时长，我们就认为 Session 已经 Timeout。因为 ZooKeeper 客户端库仅在连接重新建立的时候才抛出 timeout 的事件，所以当网络彻底隔断的情况下，需要使用前面提到的方法来发现 timeout。

###意外中断的处理
当由于网络中断使一个组件( master, slave, scheduler driver )无法联络 ZooKeeper，组件的 Master Detector 就会触发超时事件，这将通知组件已经和 leader master 失去了联系。根据组件的类型不同，可能会发生以下情况（注意：虽然组件被从 ZooKeeper 断开，但 master 仍可以与 slaves 或 schedulers 通信，反之亦然）：

- 当 slave 从 ZooKeeper 断开，并且不再知道谁是当前的 leader，它就会忽略任何 master 发来的指令。这样可以确保自己不会被非 leader 的 master 误导。当它（slave）重新连接到 ZooKeeper ，ZooKeeper 会通知它哪个是当前的 leader master。Slave 会恢复从 leader master 接收指令。

- 这个被断开的 Master 进入了无 leader 的状态，即使断开前并不是一个 leader master。

* 如果是一个 leader master 从 ZooKeeper 中断开，它将停止工作并自行销毁。用户/工程师/管理员可以重新启动一个 master 并且连接到 ZooKeeper， 它将成为一个新的备用 master。

* 如果是一个备份 masetr 被断开，那么它会等待与 ZooKeeper 重新连接，并且有可能会被选举为新的 leading master。


- 当 Scheduler drivers 从 leading master 断开，它会通知 scheduler 已经与 leader 断开连接。

当一个slave 无法再联系 leader master：

* leader 对 slave 的健康检查失败。

* leader 标记 slave 为停用的状态，并将运行在这个 slave 上的任务标记为 LOST 状态。

>本篇翻译自[http://mesos.apache.org/documentation/latest/high-availability/](http://mesos.apache.org/documentation/latest/high-availability/)
