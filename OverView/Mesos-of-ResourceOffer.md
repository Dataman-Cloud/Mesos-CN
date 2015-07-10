#Example of Resource Offer
##

**下图描述了一个 Framework 如何通过调度来运行一个 Task**




![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)
###事件流程:

1. Slave1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

**当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。**

>本篇内容翻译自[http://mesos.apache.org/documentation/latest/mesos-architecture/](http://mesos.apache.org/documentation/latest/mesos-architecture/)