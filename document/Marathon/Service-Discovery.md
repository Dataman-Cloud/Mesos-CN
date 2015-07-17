
#服务发现和负载均衡



从我们的应用程序创建和运行开始，我们就需要一种发送信息到外边的方法。如果我们当前运行多个  app， 它们也需要一种方法来找到彼此。 我们可以通过域名系统([DNS](https://en.wikipedia.org/wiki/Domain_Name_System))使用[Mesos-DNS](https://github.com/mesosphere/mesos-dns)作为服务发现。Mesos-DNS 会给正运行在  Mesos  上的每个应用程序生成一个主机名， 在当前的机器上把这些名字转化成IP地址和端口。如果通过多个 framework（不只是 Marathon）发布应用程序，Mesos-DNS是非常好用的。浏览[documentation and tutorials page](http://mesosphere.github.io/mesos-dns/)可以查看更多关于  Mesos-DNS  的信息。

替代服务发现的另一种方法是在集群中每个主机运行一个TCP代理，将本地的静态端口指向正在运行应用的主机。这样，客户只需连接到该端口就可以发现执行细节。如果这个方法就就可以满足全部的应用程序通过 Marthon 发布。

##端口分配


在Marathon上创建一个应用程序时（通过REST API或者前端），你可以分配给它一个或者多个端口。这些端口可以是任何有效的端口号或者是0，Marathon会将会随机的分配端口号。

这个端口是用于确保没有两个应用程序可以运行使用Marathon分配的重叠端口。（即这个端口是唯一的）。

然而，这不是该应用程序的每个实例的Mesos集群内对slave node上运行时被分配的端口。因为多个实例可能运行在同一个node上，每个实例（任务）分配一个随机的端口。这可以在Marathon中设置`$PORT`环境变量来访问。

每个正在运行的任务端口信息可以通过在[tasks API endpoint](https://mesosphere.github.io/marathon/docs/rest-api.html#get-/v2/tasks)中使用 `<marathon host>:<port>/v2/tasks`  查询。



##使用 HAProxy

Marathon附带一个简单的被叫做 `haproxy-marathon-bridge` 的shell脚本以及更高级的  
Python脚本 `servicerouter.py`。两个脚本都可以将Marathon的REST API列表中正在运行的任务推送到HAproxy的设置文件中，HAproxy是一个轻量级的TCP/HTTP的代理。`haproxy-marathon-bridge`提供了一个最小设置功能，对于初学者来说是容易理解的。 `servicerouter.py`支持如SSL卸载，sticky连接和虚拟主机的负载均衡的更高级的功能。


###haproxy与Marathon的桥接

通过 `haproxy-marathon-bridge`脚本从Marathon生成一个HAProxy配置在localhost:8080运行:

`$ ./bin/haproxy-marathon-bridge localhost:8080 > /etc/haproxy/haproxy.cfg`

重新加载HAProxy配置而不中断现有的连接：

    $ haproxy -f haproxy.cfg -p haproxy.pid -sf $(cat haproxy.pid)

配置脚本并重新加载可以通过Cron经常触发来跟踪topology变化。如果一个node在重新加载时消失， HAProxy的健康检查将抓住它并停止向这个node发送traffic 。

为了方便这个设置，`haproxy-marathon-bridge`  脚本以另一种方式可以调用安装脚本本身，HAProxy和定时任务每分钟ping一次的Marathon服务，如果有任何改变将立刻刷新HAProxy。


    $ ./bin/haproxy-marathon-bridge install_haproxy_system localhost:8080


- Marathon需要ping的列表存按行存储在  `/etc/haproxy-marathon-bridge/marathons`

- 脚本安装在  `/usr/local/bin/haproxy-marathon-bridge`

-cronjob安装在`/etc/cron.d/haproxy-marathon-bridge` 注意需要用root来运行。

所提供的只是一个基本的示例脚本。选项的全部列表请查看 [HAProxy configuration docs](http://cbonte.github.io/haproxy-dconv/configuration-1.5.html).



###servicerouter.py

通过`servicerouter.py`脚本从Marathon生成一个HAProxy配置在localhost:8080运行:

    $ ./bin/servicerouter.py --marathon http://localhost:8080 --haproxy-config /etc/haproxy/haproxy.cfg

如果有任何变化,将会刷新haproxy.cfg，这样HAproxy将会重新自动加载。

servicerouter.py有许多额外的功能,像sticky 会话,HTTP到HTTPS的重定向,SSL卸载,VHost支持和模板功能。

如需完整的servicerouter.py文档请运行： `console $ ./bin/servicerouter.py --help` 

