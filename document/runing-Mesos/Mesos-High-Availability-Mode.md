##Mesos 高可用性

&nbsp;&nbsp;&nbsp;&nbsp;Mesos 利用多台 Mesos master 来实现高可用性。一个 active 的 master 和 几个备份来避免意外情况的发生。 通过 Apache ZooKeeper 来协调选举出 leader ，关于 Apache ZooKeeper 原理，可以在[ Apache ZooKeeper ](http://zookeeper.apache.org/doc/trunk/recipes.html#sc_leaderElection)官网中找到。

&nbsp;&nbsp;&nbsp;&nbsp;注意：本文假定您已经知道如何运行并配置 Apache ZooKeeper，它的客户端库已经包含在标准 Mesos 中。

###用法
&nbsp;&nbsp;&nbsp;&nbsp;设置 Mesos 为高可用模式：


&nbsp;&nbsp;&nbsp;&nbsp;1. 确保 ZooKeeper 集群的启动与运行。

&nbsp;&nbsp;&nbsp;&nbsp;2. 提供所有 master ，slaver ， 调度框架的 znode 路径。如：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 使用 ***--zk*** 标志启动 mesos master二进制文件，如 ***–zk=zk://host1:port1,host2:port2,…/path'***。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 启动 mesos-slave 二进制通过 ***--master=zk://host1:port1,host2:port2,.../path***  。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 启动 framework scheduler ( 调度框架 ) 用上述两步相同的  ***zk*** 路径。 SchedulerDriver 必须使用此路径创建。

&nbsp;&nbsp;&nbsp;&nbsp;从现在开始，Mesos master 和 slaves 都与 ZooKeeper 进行交互找出当前为 leader 状态的 master 。 这是除了 leading master 与 slaves 之间外经常的通信。

###实现细节

&nbsp;&nbsp;&nbsp;&nbsp;Mesos 实现了两个层级的 leader 选举抽象，一个在 ***src/zookeeper*** ，另一个在 ***src/master*** .

&nbsp;&nbsp;&nbsp;&nbsp;- 低级的 ***LeaderContender*** 和 ***LeaderDetector*** 模仿 recipe 实现了通用的 ZooKeeper 选举算法。

&nbsp;&nbsp;&nbsp;&nbsp;- 高级的 ***MasterContender*** 和 ***MasterDetector*** 围绕 ZooKeeper 的 contender 和 detector 给抽象适配器提供/解析 ZooKeeper 数据。


&nbsp;&nbsp;&nbsp;&nbsp;- 每个 Mesos master 同时使用 contender 和 detector 选举出自己和其他 master 谁是当前的 leader 。 另外，一个单独的检测是必须的，因为每个 master 的 WebUI 定向选择的当前 leader 不是由选举产生的。其他的 Mesos组件（如 slaves和调度驱动程序）必须使用 detector（探测器）找到当前的 leader 并链接它。

&nbsp;&nbsp;&nbsp;&nbsp;leader 候选组概念在 ***Group*** 中实现。这种抽象处理可以通过 ZooKeeper 组成身份登录，注销和监控等。以下是  ZooKeeper 的几个 Session( 会话 ) 事件：

&nbsp;&nbsp;&nbsp;&nbsp;- 连接

&nbsp;&nbsp;&nbsp;&nbsp;- 重新连接

&nbsp;&nbsp;&nbsp;&nbsp;- Session 过期

&nbsp;&nbsp;&nbsp;&nbsp;- Znode 的创建，删除和更新

&nbsp;&nbsp;&nbsp;&nbsp;我们也明确 Session 的 timeout 当 ZooKeeper 从指定时间间隔中分离。 通过 ***MASTER_CONTENDER_ZK_SESSION_TIMEOUT*** 和  ***MASTER_DETECTOR_ZK_SESSION_TIMEOUT*** 这是因为 ZooKeeper 客户端库仅在重新连接  Session 通知到期。这些超时对于网络分区特别敏感。

###组件的断开处理
&nbsp;&nbsp;&nbsp;&nbsp;当由于网络分区使一个组件( master, slave, scheduler driver )从 ZooKeeper 中断开，组件的 Master Detector （ 主探测器 ）会产生超时事件。
这将通知这个组件没有 leader master 。 根据这些组件，可能会发生以下情况（注意：虽然组件被从 ZooKeeper 断开，但 master 仍可以与 slaves 或 schedulers 通信，反之亦然）：

&nbsp;&nbsp;&nbsp;&nbsp;- Slaves 从 ZooKeeper 断开并且不再知道谁是它的 leader 。它们会忽略 masters 的信息，以确保它们不会对 non-leader 的决策起作用。 当一个 slave 重新连接到 ZooKeeper ，ZooKeeper 会通知它谁是它当前的 leader，并停止它之前忽略来自  master 信息的动作。

&nbsp;&nbsp;&nbsp;&nbsp;- Masters 变成群龙无首的状态而不管它们在断开之前是否是一个  leader 。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;* 如果 leader 从 ZooKeeper 中断开，将中止其进程。 用户/工程师/管理员可以启用一个新的，连接到 ZooKeeper ， Master实例作为一个备份启动。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;* 否则，断开备份重新与 ZooKeeper 连接，并可能成为一个新的 leading master。


&nbsp;&nbsp;&nbsp;&nbsp;- Scheduler drivers 从 leading master 断开，通知 scheduler 它们已经与 leader 断开连接。

&nbsp;&nbsp;&nbsp;&nbsp;当一个网络分区从 leader 与 一个slave 断开 ：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;* slave 不能从 leader 处检测健康状态。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*  leader 标记 slave 为停用的状态，并标记它的任务为 LOST 状态。

>本篇翻译自[http://mesos.apache.org/documentation/latest/high-availability/](http://mesos.apache.org/documentation/latest/high-availability/)