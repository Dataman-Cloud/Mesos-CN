##构建运行Mesos-DNS
###构建&nbsp;Mesos-DNS
要构建Mesos-DNS,你必须在你的电脑上安装go和godep。必须设置GOPATH环境变量指向go安装包的目录，必须添加$GOPATH/bin到PATH环境变量。如果你安装go到自定义目录，需要设置GOROOT环境变量，并且把$GOROOT/bin添加到PATH环境变量中去。例如，执行以下操作：
<pre><code>export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
export GOROOT=/usr/local/go      # 假设 go 装在了 /usr/local/go 目录下
export PATH=$PATH:$GOROOT/bin
</code></pre>

用godep构建Mesos-DNS:
<pre><code>go get github.com/mesosphere/mesos-dns
cd $GOPATH/src/github.com/mesosphere/mesos-dns
make all
</code></pre>

这将生成一个静态的二进制Mesos-DNS文件,可以装在任意地方。在同一目录下你可以找到配置文件config.json。
###运行&nbsp;&nbsp;&nbsp;Mesos-DNS
若要运行Mesos-DNS,必须在所选的服务器上安装mesos-dns二进制文件。服务可以装在Mesos的任意一台机器上，或者是在同一网络的一台专用机器上。接下来，按照链接<http://mesosphere.github.io/mesos-dns/docs/configuration-parameters.html>说明为你的集群创建一个配置文件。可以这样启动Mesos-DNS
<pre><code>sudo mesos-dns -config=config.json &</code></pre>

为了加强容错能力，建议通过Marathon把Mesos-DNS服务发布到任意一台Mesos从节点上。如果Mesos-DNS失败，Marathon将重新启动它，确保几乎不间断的服务。可以通过Marathon约束从节点的主机名或者任何从节点的属性来选择把Mesos-DNS发布到哪一个从节点上面。例如下面的json描述了Marathon通过从节点的主机名发布Mesos-DNS&nbsp;&nbsp;&nbsp;&nbsp;10.181.64.13：
<pre><code>{
"cmd": "sudo  /usr/local/mesos-dns/mesos-dns -config=/usr/local/mesos-dns/config.json",
"cpus": 1.0, 
"mem": 1024,
"id": "mesos-dns",
"instances": 1,
"constraints": [["hostname", "CLUSTER", "10.181.64.13"]]
}</code></pre>

注意这个主机名字段是指从节点向Mesos注册时使用的主机名。它可能不是一个IP地址，或者是任何形式有效的主机名。可以通过Mesos的WEB页面来检查从节点的主机名属性。可以通过REST访问状态：
<pre><code>curl http://master_hostname:5050/master/state.json | python -mjson.tool</code></pre>

####设置从节点
允许Mesos任务使用Mesos-DNS作为主DNS服务，你必须在每个从节点上修改文件 /etc/resolv.conf增加新的nameserver。例如，如果mesos-dns运行的服务器地址是10.181.64.13，你应该在每个从节点/etc/resolv.conf开始加上一行nameserver 10.181.64.13。可以通过运行下面的命令实现：
<pre><code>sudo sed -i '1s/^/nameserver 10.181.64.13\n /' /etc/resolv.conf</code></pre>

如果启动多个Mesos-DNS，需要在/etc/resolv.conf的开始为每一个服务添加nameserver。这些条目的顺序将确定从节点连接Mesos-DNS实例的顺序。你可以通过设置options rotate在nameserver之间选择负载均衡的轮循机制。

/etc/resolv.conf中其他的nameserver设置保持不变。/etc/resolv.conf文件在主节点中，只需要修改同时做为从节点的机器。