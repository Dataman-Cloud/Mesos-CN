## Mesos 0.23 版本发布

Apache Mesos 0.23.0 Released

最新的Mesos版本，0.23.0，现在可以下载了。这次版本包含了以下特点和变动：

### Per-container network isolation（MESOS-1585）容器的网络隔离

Mesos 0.23 提供支持单个容器的网络监控和隔离。网络隔离避免了单个容器可变网络端口带来的烦恼，网络带宽消耗过高或者网络包传输给他人延时明显。每个运行的容器的网络统计数据是通过点 /monitor/statistics.json 形式在 slave 端点显示。网络隔离对运行在 slave 上的多数 tasks 都是明晰的（那些绑定到 port 0 和让内核分配给它们对应的端口）。此点仅仅是在 Linux 上实现并且需要一个 configure-time flag（配置时间标志位）。请参阅
 [http://mesos.apache.org/documentation/latest/network-monitoring/](http://mesos.apache.org/documentation/latest/network-monitoring/)

### SSL（MESOS-910）

对通过 libevent 的任何 libprocess 通信的 SSL 加密提供实验性支持。加密流量在 Mesos master 和它对应的 slaves 及 Frameworks 之间的信息安全是很重要的，同时它可以防止窃听和模拟。此特点需要 configure-time flag 并且有一些性能的影响。请参阅 [http://mesos.apache.org/documentation/latest/mesos-ssl](http://mesos.apache.org/documentation/latest/mesos-ssl)

### Oversubscription（MESOS-354) 过载

尝试在可撤销的资源上启动 tasks/executors，当资源被节流或抢占，Mesos 可以随时撤销任务。

为了应对峰值负荷以及一些想象不到的负载峰值，高优先级的面向用户的服务会置备在大型集群上。因此，大多数时间，资源没有得到充分的利用。过载会激发暂时没有使用的资源来最大限度的执行任务，比如背景分析，视频/图像处理，芯片仿真，以及其他低优先级的工作，当资源被征用，这些任务可以被随时撤销。

Oversubscription 增加了两个新的 slave 组成部分：资源估算和服务质量（QoS）控制，延展旁边现有的资源进行分配，资源监视器和 mesos slave。请参阅
[http://mesos.apache.org/documentation/latest/oversubscription/](http://mesos.apache.org/documentation/latest/oversubscription/)

### Persistent volumes(MESOS-1554) 持久化卷

尝试支持 Frameworks 提供创建磁盘持久化卷服务，这使得有状态服务例如 HDFS 以及 Cassandra 可以通过 Mesos 存储相关数据而不是像之前必须将数据存储在网络挂载的 EBS 卷或者将数据放置在其他位置使得磁盘资源不受管理。详情请参阅
[http://mesos.apache.org/documentation/latest/persistent-volume/](http://mesos.apache.org/documentation/latest/persistent-volume/)

### Dynamic reservations (MESOS-2018) 动态保留

在特定的 salve 上对 framework 的动态资源保留进行尝试性支持。不需要操作者在 slave 启动的时候指定一个固定的，预先计算好的“静态”保留，现在 Framework 可以保留系统提供的资源而无需重启 salve。现在没有在动态保留做出重大的改变，这意味着现有的静态预留机制继续全力支持。请参阅
[http://mesos.apache.org/documentation/latest/reservation/](http://mesos.apache.org/documentation/latest/reservation/)

### Fetcher caching（MESOS-336）缓存抓取

对 executor/task 的二进制文件读取器缓存进行尝试性支持。读取器可以被指示去缓存URI下载到一个专用的目录为后续的下载做重用。 如果对于URI的缓存被确认，缓存读取将会生效。如果 URI 是首次遇到（对于特定用户），则首先下载到高速缓存中，然后从那里复制到 sandbox 目录。如果相同的 URI 再次遇到（对于相同的用户），以及相应的高速缓存文件驻留在高速缓存或静止途中到缓存中，则不会进行下载，并且读取器直接进到从高速缓存拷贝。请参阅
[http://mesos.apache.org/documentation/latest/fetcher/](http://mesos.apache.org/documentation/latest/fetcher/)

### 注：实验状态

SSL， Oversubscription， Persisteng volumes， Dynamic reservations ，以及 Fetcher caching 现在均处于试验阶段，它们是功能完整，但是在一些使用情况，还没有在生产环境中进行测试。
