#Example of Resource Offer

***下图描述了一个Framework如何通过调度来运行一个Task***




![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)
###事件流程:

1. Slave1向Master报告，有4个CPU和4GB内存可用
2. Master发送一个Resource Offer给Framework1来描述Slave1有多少可用资源
3. FrameWork1中的FW Scheduler会答复Master，我有两个Task需要运行在Slave1，一个Task需要<2个CPU，1GB内存>，另外一个Task需要<1个CPU，2GB内存>
4. 最后，Master发送这些Tasks给Slave1。然后，Slave1还有1个CPU和1GB内存没有使用，所以分配模块可以把这些资源提供给Framework2

**当Tasks完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。**

***本篇内容翻译自[http://mesos.apache.org/documentation/latest/mesos-architecture/](http://mesos.apache.org/documentation/latest/mesos-architecture/)***