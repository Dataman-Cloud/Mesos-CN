## Mesos 简介

**Mesos —— 像用一台电脑（一个资源池）一样使用整个数据中心**

### Mesos是什么?

**分布式操作系统内核**

Mesos是以与Linux内核同样的原则而创建的，不同点仅仅是在于抽象的层面。Mesos内核运行在每一个机器上，同时通过 API 为各种应用提供跨数据中心和云的资源管理调度能力。这些应用包括Hadoop、Spark、Kafka、Elastic Search。还可配合框架 Marathon 来管理大规模的Docker等容器化应用。

### Mesos特性

- 可扩展到10000个节点
- 使用 ZooKeeper 实现 Master 和 Slave 的容错
- 支持 Docker 容器
- 使用 Linux 容器实现本地任务隔离
- 多资源调度能力（内存，CPU、磁盘、端口）
- 提供 Java，Python，C++等多种语言 APIs
- 通过 Web 界面查看集群状态
- 新版本将支持更多功能

>本篇内容翻译自[http://mesos.apache.org/](http://mesos.apache.org/)
