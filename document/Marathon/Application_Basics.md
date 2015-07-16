#Application Basics
#基本应用

应用在Marathon中是一个综合的概念。每个应用程序通常代表一个长期运行的服务,其中包括多个实例同时运行在多台主机上。

##Hello Marathon：内嵌的shell脚本

让我们从一个简单的例子开始，一个app每隔5秒不断的向终端输出Hello Marathon，使用JSON语言格式，您应该像下面这段代码一样来描述这个应用。

    {
    "`id`": "`basic-0`", 
    "`cmd`": "`while [ true ] ; do echo 'Hello Marathon' ; sleep 5 ; done`",
    "`cpus`": `0.1`,
    "`mem`": `10.0`,
    "`instances`":` 1`
    }

需要注意的是在上面例子中的cmd是可执行命令。它是通过Mesos底层命令`/bin/sh -c ${cmd}`来执行。
![](https://mesosphere.github.io/marathon/img/marathon-basic-0.png)



当Marathon将应用执行权限交给Mesos，Mesos在自有的Sandbox环境中执行任务。Sandbox在slave上是一个特殊的目录，充当运行环境，同时包含相关的日志文件, 以及命令执行中的产生的stderr和stdout。在[debugging distributed apps](http://open.mesosphere.com/tutorials/debugging-a-mesosphere-cluster/)中可以找到sandbox的相关介绍。


##在应用程序中使用资源


运行一些重要的应用你通常需要依赖资源的合集，比如文档。对于这种情况, Marathon有URl的规定格式。在下载解压资源的方面,充分利用了Mesos抓取的长处。



在我们深入这个话题之前，让我们先来看一个例子：

    {
    "`id`": "`basic-1`", 
    ， `cmd`": "`./cool-script.sh`",
    "`cpus`": `0.1`,
    "`mem`": `10.0`,
    "`instances`": `1`,
    "`uris`": [
        "`https://example.com/app/cool-script.sh`"
    ]
    }




上面的例子做了什么：在执行`cmd`之前, 先从`https://example.com/app/cool-script.sh`下载资源（通过Mesos), 使其在应用测试的沙箱中是可用的。可以检查这些已下载通过访问Mesos UI, 可以点击进入Mesos工作节点的沙箱中，你现在应该找到`cool-script.sh`。



注意的是Mesos版本v0.22或更高抓取代码, 不会进行下载可执行文件在默认情况下。因此`cmd `中应该是`chmod u+x cool-script.sh && ./cool-script.sh`。




如上面已经提到的, Marathon也知道如何处理应用资源。当前, Marathon将（通过Mesos执行`cmd`之后）首先尝试解压和提取资源具有以下文件拓展名字：

- .tgz
- .tar.gz
- .tbz2
- .tar.bz2
- .txz
- .tar.xz
- .zip



如何来实践表明通过下面的例子：让我们假设你有一个应用程序在一个zip中，其地址是`https://example.com/app.zip`。这个zip压缩文件中包含脚本`cool-script.sh`, 这就是你要执行什么。

    {
    "`id`": "`basic-2`", 
    "`cmd`": "`app/cool-script.sh`",
    "`cpus`": `0.1`,
    "`mem`": `10.0`,
    "`instances`": `1`,
    "`uris`": [
        "`https://example.com/app.zip`"
    ]
    }



需要注意的是对比实例`basic-1`, 我们现在有一个`cmd`内容是`app/cool-script.sh`。事实上, 当压缩文件被下载并提取, 目录`app`根据文件名`app.zip`被创建, zip文件的内容提取到。


另请注意的是, 你也可同时指定多个资源, 不是仅仅就一个。所以, 例如, 你可以提供从CDN一个Git仓库和一些资源如下：

    {
    ...
    "uris": [
        "https://git.example.com/repo-app.zip", "https://cdn.example.net/my-file.jpg", "https://cdn.example.net/my-other-file.css"
    ]
    ...
    }


在典型的开发和部署周期中，自动化构建系统将应用程序二进制文件的位置是通过一个URI下载。Marathon能够下载资源从许多的来源。支持以下[URI schemes](http://tools.ietf.org/html/rfc3986#section-3.1):



- file:
- http:
- https:
- ftp:
- ftps:
- hdfs:
- s3:


##一个简单的Docker-based应用


使用Docker镜像的应用程序可以直接通过Marathon运行。可以通过[Running Docker Containers on Marathon](https://mesosphere.github.io/marathon/docs/native-docker.html)了解进一步的细节和高级选项。



在下面的例子中的应用程序定义中, 我们将重点在一个简单的Docker的app：一个Python-based的web服务使用[python:3](https://registry.hub.docker.com/_/python/)镜像。在container里边， 这个web服务运行的端口是`8080`（`containerPort`的值）。在container外，Marathon会分配一个随机端口（`hostPort`设置是0）。



    {
      "id": "basic-3",
      "cmd": "python3 -m http.server 8080",
      "cpus": 0.5,
      "mem": 32.0,
      "container": {
    "type": "DOCKER",
    "docker": {
      "image": "python:3",
      "network": "BRIDGE",
      "portMappings": [
    { "containerPort": 8080, "hostPort": 0 }
      ]
    }
      }
    }



Marathon的UI目前还不支持直接启动Docker-based应用程序, 我们将使用[HTTP API](https://mesosphere.github.io/marathon/docs/rest-api.html)来部署`basic-3`的应用。

    curl -X POST http://10.141.141.10:8080/v2/apps -d @basic-3.json -H "Content-type: application/json"



假设，你粘贴示例JSON到一个文件命名为`basic-3.json`的文件而且你正在使用[playa-mesos](https://github.com/mesosphere/playa-mesos)，一个基于Vagrant的Mesos sandbox的测试部署环境。当将上面的应用提交至Marathon，应该学习一些关于Marathon UI的知识（关于任务和配置选项）

![](https://mesosphere.github.io/marathon/img/marathon-basic-3-tasks.png)
![](https://mesosphere.github.io/marathon/img/marathon-basic-3-config.png)

这些例子的结果是，Marathon已经在Docker容器中启动了一个基于Python-based的web服务， 容器的root文件夹地址在` http://10.141.141.10:31000`。


