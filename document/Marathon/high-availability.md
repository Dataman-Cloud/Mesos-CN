#介绍

Marathon默认支持高可用性的操作模式, 当单个实例不可用时应用程序可以继续运行。这是通过在ZooKeeper下管理着多个Marathon来实现的，如果当前Marathon主节点故障,ZooKeeper将根据选举机制选出新的Marathon主节点。



#配置


当`--ha`命令行参数设置成`true`的时Marathon启动高可用性模式。其默认值是true，所以不需要单独设置。



每个Marathon必须指定同样的Zookeeper节点的个数，例如，如果你的`zk://1.2.3.4:2181,2.3.4.5:2181,3.4.5.6:2181/marathon`，以正常的方式启动每个实例， 

  `--zk zk://1.2.3.4:2181,2.3.4.5:2181,3.4.5.6:2181/marathon`

#代理

与Mesos的web控制台不同，Marathon的web控制台不会重新定向到当前的Marathon主节点。它将请求进行委托，所以在Web控制台显示的数据将是应用程序的当前运行状态。
Proxy的机制也适用于向Marathon的REST API提出请求的过程。

