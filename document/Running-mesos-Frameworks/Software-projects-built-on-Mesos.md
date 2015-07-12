##建立在 Mesos 上的软件项目

###常驻服务
- [Aurora ](http://aurora.apache.org/)是一个运行在 Mesos之上，使你能够运行一些常驻服务，充分利用了 Mesos 的可扩展性、容错性以及资源隔离的服务调度程序。
- [Marathon ](https://github.com/mesosphere/marathon)是建立在 Mesos 上的私有 PaaS。它会自动处理硬件或者软件故障，并确保一个应用程序是"永远在线"的状态。
- [Singularity ](https://github.com/HubSpot/Singularity)是一个调度 Mesos 任务的任务调度器( HTTP API 和 Web 接口)，包括：长时间运行的进程，一次性的任务，调度作业。
- [SSSP ](https://github.com/mesosphere/sssp)是一个简单的 Web应用程序，它提供了" Megaupload "用于在 S3 中储存和共享文件。

###大数据处理
- [Cray Chapel ](https://github.com/nqn/mesos-chapel)是一个高效的并发编程语言。它可以让你在 Mesos 上运行 Chapel 项目。
- [Dpark ](https://github.com/douban/dpark)是一个用 Python 编写的 Spark 克隆版，该计算框架类似于 MapReduce。
- [Exelixi ](https://github.com/mesosphere/exelixi)是一个运行遗传算法的分布式框架
- [Hadoop ](https://github.com/mesos/hadoop)在 Mesos 上运行  Hadoop 可以有效的在整个集群中分配 MapReduce 任务。
- [Hama ](http://wiki.apache.org/hama/GettingStartedMesos)是一个基于 BSP ( Bulk Synchronous Parallel )的分布式计算框架，用来处理大规模的科学计算,如矩阵和图计算。
- [MPI ](https://github.com/mesosphere/mesos-hydra)是一个消息传递系统，在各式各样的并行计算机中传递消息。
- [Spark ](http://spark.apache.org/)是一个快速并且通用的处理大规模数据的引擎
- [Storm ](https://github.com/mesosphere/storm-mesos)是一个分布式实时计算系统。Storm 大大简化了面向庞大规模数据流的处理机制，从而在实时处理领域扮演着 Hadoop 之于批量处理领域的重要角色。

###批处理调度
- [Chronos ](https://github.com/mesos/chronos)是一个分布式作业调度程序，支持复杂的拓部结构。可以更加容错来替代 Crom。
- [Jenkins ](https://github.com/jenkinsci/mesos-plugin)是一个持续集成服务。 mesos-jenkins 插件允许在集群中根据工作量动态启动 work。
- [JobServer ](http://www.grandlogic.com/content/html_docs/products.shtml#jobserverprod)是一个分布式作业调度和处理器，它允许开发人员创建自定义的批处理任务通过 web UI。

###数据储存
- [Cassandra ](https://github.com/mesosphere/cassandra-mesos)是一个高可靠的大规模分布式存储系统。高度可伸缩的、一致的、分布式的结构化 key-value 存储方案，线性可扩展性以及在商业硬件及云基础上的容错系统，使其成为关键任务数据的理想平台。
- [ElasticSearch ](https://github.com/mesosphere/elasticsearch-mesos)是一个分布式搜索引擎，Mesos 使其很容易的运行和部署。
- [Hypertable ](https://code.google.com/p/hypertable/wiki/Mesos)是一个高性能、可扩展的分布式结构化和非结构化数据的存储和处理系统。

>本篇内容翻译自http://mesos.apache.org/documentation/latest/mesos-frameworks/
