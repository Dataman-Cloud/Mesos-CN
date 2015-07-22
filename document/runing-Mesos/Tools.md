#工具

##运维工具

使用这些工具可以很容易地建立和运行 Mesos 集群。


- [collectd plugin](https://github.com/rayrod2030/collectd-mesos) 收集 Mesos 集群指标。

- [Deploy scripts](http://mesos.apache.org/documentation/latest/deploy-scripts/) 在一组机器上发布  Mesos  集群。

- [Chef cookbook by Everpeace](https://github.com/everpeace/cookbook-mesos)安装  Mesos  和配置  master  和  slave 节点。这个 cookbook 支持从 Mesosphere的包或者源代码安装。

- [Chef cookbook by Mdsol](https://github.com/mdsol/mesos_cookbook) 用来安装  Apache  mesos 集群管理器的 application cookbook。  通过这个 cookbook 可以从 Mesosphere 的包来安装 Mesos。

- [Puppet Module by Deric](https://github.com/deric/puppet-mesos)  用来管理集群中的 Mesos 节点的 Puppet 模块。

- [ Vagrant setup by Everpeace](https://github.com/everpeace/vagrant-mesos) 通过  Vagrant 运行  Mesos  集群。

- [ Vagrant setup by Mesosphere](https://github.com/mesosphere/playa-mesos)使用 Vagrant 快速建立 Mesos sandbox 环境。


##开发人员工具

如果你想破解  Mesos 或者自己写一个新的 framework，先看看下面的 scheduler 和 executor 例子：

- [Go Bindings and Examples](https://github.com/mesosphere/mesos-go) Go 语言的 Mesos 框架开发接口，并且包含用 Go 写的 scheduler 和 executor 例子。

- [ Mesos Framework giter8 Template](https://github.com/mesosphere/scala-sbt-mesos-framework.g8) 这是一个 giter8 模板，可以创建一个使用  Scala 语言编写的 Mesos 的框架。除了使用 SBT 作为开发接口，你还可以用 Vagrant 来启动一个单机环境进行测试。

- [Scala Hello World](https://gist.github.com/guenter/7471695) 一个简单的  Mesos "Hello World": 下载并且在集群每个节点上都启动一个 web 服务器。

- [Xcode Workspace](https://github.com/tillt/xcode-mesos) 让 Xcode Workspace 支持 Mesos。

Can’t find yours in the list? Please submit a patch, or email user@mesos.apache.org and we’ll add you!

