
#设置与运行Marathon


##快速开始


让我们用一个简单的方法通过虚拟机安装Mesos和Marathon在本地执行[playa-mesos](https://github.com/mesosphere/playa-mesos)项目。



###软件需求
- [Apache Mesos](https://mesos.apache.org/ "Apache Mesos") 0.20.1+
- [Apache ZooKeeper](https://zookeeper.apache.org/)
- JDK 1.6+
- Scala 2.11+
- sbt 0.13.5


##安装过程


###安装Mesos


一个简单的方法是通过系统的包管理来安装。如何在主要的Linux发型板以及Mac OS X上安装的资料可以在我们的[下载界面](https://mesosphere.com/downloads/)中找到。
如果打算从源码建立mesos，可以参考Mesos [Getting Started](http://mesos.apache.org/gettingstarted/).使用`make install`命令在`/usr/local`中安装Mesos,使用命令行和使用包安装是一样的。


###安装Marathon
通过Package Manager，Marathon可以从软件仓库中下载。
对于 Mesos 0.22.1+：

1. 下载最新版本的Marathon

    $ curl -O http://downloads.mesosphere.com/marathon/v0.8.2/marathon-0.8.2.tgz
    $ tar xzf marathon-0.8.2.tgz




对于Mesos 0.20.0~0.22.0：


请考虑讲Mesos升级到0.22.1！

    $ curl -O http://downloads.mesosphere.com/marathon/v0.8.1/marathon-0.8.1.tgz
    $ tar xzf marathon-0.8.1.tgz



对于Mesos 0.19.0：

    $ curl -O http://downloads.mesosphere.com/marathon/marathon-0.6.1/marathon-0.6.1.tgz
    $ tar xzf marathon-0.6.1.tgz
    



对于Mesos 0.17.0~0.18.2：
    
    $ curl -O http://downloads.mesosphere.com/marathon/marathon-0.5.1/marathon-0.5.1.tgz
    $ tar xzf marathon-0.5.1.tgz



对于Mesos 0.16.0乃至更早的版本：
    
    $ curl -O http://downloads.mesosphere.com/marathon/marathon-0.5.1_mesos-0.16.0/marathon-0.5.1_mesos-0.16.0.tgz
    $ tar xzf marathon-0.5.1_mesos-0.16.0.tgz



通过将`.sha25`6加在Url上可以进行`SHA-256`校验。



###版本相关


从版本0.9.0开始Marathon将坚持语义版本。这意味着我们致力于保持我们的REST API跨版本兼容性，除非我们改变主要版本（版本号的第一个数字）。
如果您实用的是没有被记录的特性，请通过GitHub告知我们。
标记为EXPERIMENTAL的API不包含在这个规则之内，同时我们不会在PATCH版本中加入新特性的介绍。

我们可能会在极少数的次版本升级的时候改变Marathon服务进程的命令行界面。请随时查看各个版本说明。




###升级至更新的版本


将Marathon升级至更新的版本是无缝的。


我們一般建议在升级到下一个版本的时候，创建一个ZooKeeper的备份。通过复制ZooKeeper的数据文件夹来实现被备份。


####从0.7系列版本升级至0.8系列版本或从0.8系列升级至0.9系列



0.8与0.9系列版本仅仅增加新可选字段，并没有改变存储格式。因此，升级并不需要做迁移。
只要你不能启动新特性，你可以在任何出现错误的时候回滚。尽管如此，也不要忘记对ZooKeeper的状态做一个备份。


####从0.6版本至0.7版本的升级


现在已经无法从0.7.0版本降级到更老的版本了，因为我们已经在数据格式上做出了改变。可以从这里找到升级指导。

在产品模式下运行


在产品模式下启动Marathon，你需要ZooKeeper和Mesos同时运行，下面的命令将会在产品模式中启动Marathon，将你的web浏览器的localhost设为8080，你将会看到Marathon的界面。
    
    $ ./bin/start --master zk://zk1.foo.bar:2181,zk2.foo.bar:2181/mesos --zk zk://zk1.foo.bar:2181,zk2.foo.bar:2181/marathon




Marathon使用`--master`命令去发现Mesos的master节点，使用`--zk`发现ZooKeeper，因为他们相对独立，所以Mesos master也使用其他方式发现。


对于所有的设置选项，请使用命令行[command line flags](https://mesosphere.github.io/marathon/docs/command-line-flags.html) doc。对于Marathon更多的高可用特性，请查阅[high-availability](https://mesosphere.github.io/marathon/docs/high-availability.html) doc

####Mesos Library

通过`bin/start`可以找到普遍的安装路径，`/usr/lib`和`/usr/local/lib`是Mesos自带的链接库，如果你为链接库设置了其他路径，可以将`MESOS_NATIVE_JAVA_LIBRAR`的全路径设为环境变量，设置如下。


例如：

    $ MESOS_NATIVE_JAVA_LIBRARY=/Users/bob/libmesos.dylib ./bin/start --master local --zk zk://localhost:2181/marathon
    

###启动应用


Marathon应用的介绍以及如何执行，请查阅[Application Basics](https://mesosphere.github.io/marathon/docs/application-basics.html)。