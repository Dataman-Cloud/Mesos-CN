##建立在 Mesos 上的软件项目

###常驻服务
- [Aurora ](http://aurora.apache.org/)是一个运行在 Mesos之上的服务调度程序。它能充分利用了 Mesos 的可扩展性、容错性以及资源隔离，让你能够运行常驻服务。
- [Marathon ](https://github.com/mesosphere/marathon)是建立在 Mesos 上的私有 PaaS平台。它能自动处理硬件或者软件故障，并确保每个应用程序都"永远在线"。
- [Singularity ](https://github.com/HubSpot/Singularity)是一个调度 Mesos 任务的任务调度器( HTTP API 和 Web 接口)，包括：长时间运行的进程，一次性的任务和调度作业。
- [SSSP ](https://github.com/mesosphere/sssp)是一个简单的 Web应用程序，它提供了" Megaupload "功能，用于在 S3 中储存和共享文件。

###大数据处理
- [Cray Chapel ](https://github.com/nqn/mesos-chapel)是一个高效的并发编程语言。Chapel 调度器可以让你在 Mesos 上运行 Chapel 项目。
- [Dpark ](https://github.com/douban/dpark)是一个用 Python 编写的克隆版 Spark，该计算框架类似于 MapReduce。
- [Exelixi ](https://github.com/mesosphere/exelixi)是一个海量运行遗传算法的分布式框架
- [Hadoop ](https://github.com/mesos/hadoop)在 Mesos 上运行 Hadoop。 它可以有效地在整个集群中分配 MapReduce 任务。
- [Hama ](http://wiki.apache.org/hama/GettingStartedMesos)是一个基于 BSP ( Bulk Synchronous Parallel )的分布式计算框架，用来处理大规模的科学计算,如矩阵和图计算。
- [MPI ](https://github.com/mesosphere/mesos-hydra)是一个消息传递系统，在各式各样的并行计算机中传递消息。
- [Spark ](http://spark.apache.org/)是一个快速并且通用的处理大规模数据的引擎
- [Storm ](https://github.com/mesosphere/storm-mesos)是一个分布式实时计算系统。Storm 大大简化了面向庞大规模数据流的处理机制，从而在实时处理领域扮演着 Hadoop 之于批量处理领域的重要角色。

###批处理调度
- [Chronos ](https://github.com/mesos/chronos)是一个分布式作业调度程序，支持复杂的拓部结构。它可以成为 cron 的替代品，提供更好的容错性。
- [Jenkins ](https://github.com/jenkinsci/mesos-plugin)是一个持续集成服务。 根据集群的任务内容，mesos-jenkins 插件可以动态地启动 worker。
- [JobServer ](http://www.grandlogic.com/content/html_docs/products.shtml#jobserverprod)是一个分布式作业调度和处理器。开发人员通过在 Web UI 上进行拖拽，就能创建自定义的批处理任务。

###数据储存
- [Cassandra ](https://github.com/mesosphere/cassandra-mesos)是一个高可靠的大规模分布式存储系统。它具有线性扩展性以及在商业硬件和云端体现的容错性，成为承载关键任务数据的理想平台。
- [ElasticSearch ](https://github.com/mesosphere/elasticsearch-mesos)是一个分布式搜索引擎，Mesos 让它的运行和部署变地非常容易。
- [Hypertable ](https://code.google.com/p/hypertable/wiki/Mesos)是一个高性能、可扩展的存储和处理系统，主要用来处理分布式结构化和非结构化数据。

>本篇内容翻译自http://mesos.apache.org/documentation/latest/mesos-frameworks/
