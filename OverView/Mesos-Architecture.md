###Mesos Architecture

![Mesos](http://mesos.apache.org/assets/img/documentation/architecture3.jpg "Title")

上图显示了 Mesos 的主要组成部分。 Mesos 由一个 master daemon 来管理 slave daemon 在每个集群节点上的运行, mesos applications （ 也称为 frameworks ）在这些 slaves 上运行 tasks。

Master 使用 Resource Offers 实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。 master 根据指定的策略来决定分配多少资源给 framework ，如公平共享策略，或优先级策略。 为了支持更多样性的策略，master 采用模块化结构，这样就可以方便的通过插件形式来添加新的分配模块。

在 Mesos 上运行的 framework 由两部分组成：一个是 scheduler ，通过注册到　master 来获取集群资源。另一个是在 slave 节点上运行的 executor 进程，它可以执行 framework 的 task 。 Master 决定为每个　framework 提供多少资源， framework 的 scheduler 来选择其中提供的资源。当 framework 同意了提供的资源，它通过 master 将 task发送到提供资源的　slaves 上运行。

###资源供给的一个例子

**下图描述了一个 Framework 如何通过调度来运行一个 Task**




![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)
####事件流程:

1. Slave1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

**当 Tasks 完成和有新的空闲资源时，Resource Offer 会不断重复这一个过程。**
当 Mesos 提供的瘦接口允许其来扩展和允许 frameworks 相对独立的参与进来，一个问题将会出现： 一个 framwork 的限制如何被满足在不被 Mesos 对这些限制所知晓的情况下？ 例如， 一个 framework 如何得到数据本地化在不被 Mesos所知晓哪个节点存储着被该 framwork 所需要的数据？Mesos 通过简单的寄予 frameworks 能够拒绝 offers 的能力来回答了这个问题。 一个 framework 将拒绝 不满足其限制要求的 offers 并接受满足其限制要求的 offers. 特殊情况下，我们找到一个简单的策略 delay scheduling， 在该 frameworks 等待 一个限制时间来获取存储输入数据的节点， 并生成接近的优化过得数据点。

你也可以从这里了解更多的 Mesos 架构：[Mesos技术文档](http://mesos.berkeley.edu/mesos_tech_report.pdf)

>本篇内容翻译自[http://mesos.apache.org/documentation/latest/architecture/](http://mesos.apache.org/documentation/latest/architecture/)
