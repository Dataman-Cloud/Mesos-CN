###Mesosphere Release Mesos-DNS Service Discovery for Apache Mesos

&nbsp;&nbsp;&nbsp;&nbsp;Mesosphere 已经发布了 alpha 版本的 Mesos-DNS,它是一个开源的服务发现框架用来发现运行在 Mesos 上的应用程序。

&nbsp;&nbsp;&nbsp;&nbsp;Apache Mesos 是一个开源的集群管理器，用来抽象 CPU ,内存，储存等计算资源，并且支持容错以及弹性分布式系统。 Mesos 内核运行在每个集群机器中，并且提供为应用程序提供 API 来管理集群和调度。

&nbsp;&nbsp;&nbsp;&nbsp;Apache Mesos 由一个 master 守护进程管理每个集群节点上运行的 slave 守护进程, Mesos 应用程序(也称为 frameworks)运行在这些 slaves 上。例如用来处理大数据的 Apache Spark 和 Hadoop ，常驻服务 Mesosphere 的 Marathon 和 Apache 的 Aurora 。 由于资源的动态分配等原因，这些服务通常在这样的环境中是不容易被找到的。

&nbsp;&nbsp;&nbsp;&nbsp;新开源的 Mesos-DNS 框架支持服务发现在 Apache Mesos 集群中，允许应用程序和服务通过域名系统( DNS )来相互定位。 Mesos-DNS 充当的角色和在互联网中 DNS 的作用差不多。

&nbsp;&nbsp;&nbsp;&nbsp;有 Marathon 或者 Aurora 框架启动的应用程序或者常驻服务被赋予名字，如 search.marathon.mesos 或者 log-aggregator.aurora.mesos ，Mesos-DNS 将每个集群中正在运行的应用程序的名字翻译成IP地址和端口号。这使得任何链接到集群中运行的服务都可以通过 DNS 进行查找。

&nbsp;&nbsp;&nbsp;&nbsp;Mesos-DNS 被设计成简单并且无状态的，它不需要共识机制，永久储存以及日志。这是可以的，因为 Mesos-DNS 没有实现心跳，状态监测，或者管理应用方程序的生命周期。这些功能在 Mesos master, slaves, 和 frameworks 中实现。

***Mesos-DNS在Github中的结构图：***

![Mesos-DNS](/pic/Mesos-DNS.png)

&nbsp;&nbsp;&nbsp;&nbsp;Frameworks运行在 Mesos 中，不需要直接与 Mesos-DNS 沟通。 Mesos-DNS 定期查询 Mesos master(s) ，检索所有 running 框架中正在运行的任务的状态，并为这些任务((A and SRV records)生成 DNS 记录。当 Mesos 集群中的任务开始，结束，或重启， Mesos-DNS 都需要更新 DNS 记录以保证为最新状态。

&nbsp;&nbsp;&nbsp;&nbsp;Mesos-DNS 通过 Mesos master(s) 制定的配置来运行，可以容错，并且可以通过一个框架，像 Marathon 可以监视应用程序的健康和重启它。从故障中重启后， Mesos-DNS 检索 Mesos master(s)和serves 的最新状态，并提供 DNS 请求。

&nbsp;&nbsp;&nbsp;&nbsp;Christos Kozyrakis，Mesos-DNS的核心工作者称，我们有两种动机来实现 Mesos-DNS ,而不是像其他的 DNS 系统，如 SkyDNS 或者 Consul 。首先，我们需要一个 DNS 系统来密切配合 Mesos , 而不是让每个用户或者框架描述两次任务(一次 Mesos执行，一次到 DNS系统)，它更容易和简洁的从 Mesos 传递信息到 DNS 。其次，我们需要一个解决方法。 Mesos及其框架已经实现了容错性和生命周期的管理。我们不想强迫 Mesos用户再为 DNS 部署另一套共识机制，永久储存或日志。

&nbsp;&nbsp;&nbsp;&nbsp;Mesos-DNS的 alpha 版本支持应用程序命名的基本方案。 Mesosphere博客中指出，即将到来的新版本将支持更灵活的命名规则。

&nbsp;&nbsp;&nbsp;&nbsp;Mesos-DNS 也将延伸到 Mesos 引入的安全和网络方面，并集成到即将到来的 Mesosphere Datacenter Operating System (DCOS) 中，以支持在公有云，私有数据中心以及混合部署中的服务发现。

