## 什么是Mesos

Apache Mesos是由加州大学伯克利分校的AMPLab首先开发的一款开源群集管理软件，支持 Hadoop、ElasticSearch、Spark、Storm 和 Kafka 等应用架构。

## Mesos特性

- 可扩展到10000个节点
- 使用 ZooKeeper 实现 Master 和 Slave 的容错
- 支持 Docker 容器
- 使用 Linux 容器实现本地任务隔离
- 基于多资源（内存，CPU、磁盘、端口）调度
- 提供 Java，Python，C++等多种语言 APIs
- 通过 Web 界面查看集群状态
- 新版本将支持更多

## 架构说明
### Mesos架构图
![Mesos](http://mesos.apache.org/assets/img/documentation/architecture3.jpg "Title")

### 说明
* Mesos本身包含两个组件:Master Daemon和Slave Daemon。
    * Master Daemon
        * 管理所有的 Slave Daemon。
        * 用[Resource Offers](https://github.com/Dataman-Cloud/Mesos-CN/blob/master/OverView/Mesos-of-ResourceOffer.md)实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。
        * 限制和提供资源给应用框架使用。
        * 使用可拔插的模块化的架构，方便增加新的策略控制机制。
    * Slave Daemon
        * 负责接收和管理 Master 发来的需求 Task
        * 支持使用各种环境运行各种 Task，如Docker、VM、进程调度(纯硬件)。
        
* Mesos上的task由2个组件管理:调度器(Scheduler)和执行进程(Executor Process)
    * 调度器(Scheduler)
        * 调度器通过注册 Mesos Master获得集群资源调度权限
        * 调度器可通过 MesosSchedule Driver 接口和 Mesos Master 交互
    * 执行进程(Executor Process)
        * 用于启动框架内部的 Task
        * 不同的调度器使用不同的 Executor

* Mesos 集群为了避免单点故障，所以使用 Zookeeper 进行集群交互。
