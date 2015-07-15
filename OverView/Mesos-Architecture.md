###Mesos Architecture

![Mesos](http://mesos.apache.org/assets/img/documentation/architecture3.jpg "Title")

上图显示了 Mesos 的主要组成部分。 Mesos 由一个 master daemon 来管理 slave daemon 在每个集群节点上的运行, mesos applications （ 也称为 frameworks ）在这些 slaves 上运行 tasks。

Master 使用 Resource Offers 实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。 master 根据指定的策略来决定分配多少资源给 framework ，如公平共享策略，或优先级策略。 master 采用热插拔的方式实现了模块，为了以后更好的扩展。


在 Mesos 上运行的 framework 由两部分组成：一个是 scheduler ，通过注册到　master 来获取集群资源。另一个是在 slave 节点上运行的 executor 进程，它可以执行 framework 的 task 。 Master 决定为每个　framework 提供多少资源， framework 的 scheduler 来选择其中提供的资源。当 framework 同意了提供的资源，它通过 master 将 task发送到提供资源的　slaves 上运行。

###Example of Resource Offer

**下图描述了一个 Framework 如何通过调度来运行一个 Task**




![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)
####事件流程:

1. Slave1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

**当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。**

While the thin interface provided by Mesos allows it to scale and allows the frameworks to evolve independently, one question remains: how can the constraints of a framework be satisfied without Mesos knowing about these constraints? For example, how can a framework achieve data locality without Mesos knowing which nodes store the data required by the framework? Mesos answers these questions by simply giving frameworks the ability to reject offers. A framework will reject the offers that do not satisfy its constraints and accept the ones that do. In particular, we have found that a simple policy called delay scheduling, in which frameworks wait for a limited time to acquire nodes storing the input data, yields nearly optimal data locality.

>本篇内容翻译自[http://mesos.apache.org/documentation/latest/mesos-architecture/](http://mesos.apache.org/documentation/latest/mesos-architecture/)