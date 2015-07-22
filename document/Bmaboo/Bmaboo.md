# bamboo



![](https://cloud.githubusercontent.com/assets/37033/4110258/a8cc58bc-31ef-11e4-87c9-dd20bd2468c2.png)

Bamboo是一个web守护进程其自动化的配置haproxy用于部署在[Apache Mesos](http://mesos.apache.org/)和[Marathon](https://mesosphere.github.io/marathon/)之上的web服务。  
它的功能包含：  
用来配置针对每个Marathon应用配置的HAProxy ACL规则的用户接口。   
用来配置proxy ACL规则的Rest API  
基于你的模板来自动配置HAProxy的配置文件；你可以在应用中规定你的自有模板来启用SSL和HAProxy状态接口 ，或配置不同的负载均衡策略 
可选的处理健康监测终端如果Marathon应用和Healthchecks一起被配置  
守护进程自身是无状态的；便于横向复制和扩展  
使用Golang开发，部署于HAProxy实例之上无任何附加依赖需求  
可选的于StatsD集成来监控配置重加载事件


**兼容性  **  

v0.1.1 支持Marathon 0.6和Mesos 0.19.x   
v0.2.2 支持DNS和非DNS代理ACL规则  
v0.2.8 支持HTTP&TCP通过定制Marathon环境变量  
v0.2.9 支持Marathon 0.7.* （http_callback被启用的状态下）和Mesos 0.21.X  
v0.2.11 升级了API，失效了之前的API端点


**发布和更改记录 ** 

自从Marathon API 和特性在过去已经做了更改，尤其最近几天。你需要期望我们旨在扑捉到这些更改，来提高设计并添加新的功能。我们旨在尽可能地维护向后兼容。发布和更改记录请参考[发布页面](https://github.com/QubitProducts/bamboo/releases)当需要升级 的时候。  

**部署指导**  

你可以在每个Mesos从节点上部署Bamboo和HAProxy.每个web服务将分配在Mesos从节点可以通过ACL规则生成的你指定的localhost或域来发现服务。替代式的，你可以通过不素Bamboo和HAPRoxy在不同的实例上器意味着你需要负载均衡HAProxy集群。  
![bamboo和HAPRroxy架构](https://cloud.githubusercontent.com/assets/37033/4110199/a6226b8e-31ee-11e4-9734-68e0da00767c.png)

**bambo设置指导 **

**用户接口  **

UI在管理和可视化代理规则的当前状态时非常有效。因此，你可以通过通过配置HAProxy模板来负载均衡Bamboo.  

![用户接口列表](https://cloud.githubusercontent.com/assets/37033/4320901/527988dc-3f3b-11e4-8672-666605eb1ddf.png)

![用户接口](https://cloud.githubusercontent.com/assets/37033/4320873/2ac65c48-3f3b-11e4-969e-52381dd33aae.png)


**StatsD Monitoring**

![StatsD监控](https://cloud.githubusercontent.com/assets/37033/4117219/cef5cea2-328e-11e4-8346-ecc4e4e6046b.png)

**配置和模板**  

Bamboo程序接受-config选项来指定应用配置JSON文件位置。输入-help来得到当前可用的选项。 

示例配置和HAProxy模板可以在[production.example.json](https://github.com/Dataman-Cloud/bamboo/blob/master/config/production.example.json)和[haproxy_template.cfg](https://github.com/Dataman-Cloud/bamboo/blob/master/config/haproxy_template.cfg)中找到。此章节尝试解释以代码方式来解释如何使用。


    {
        // Marathon instance configuration
        "Marathon": {
            // Marathon service HTTP endpoint
            "Endpoint": "http://localhost:8080"
    },

    "Bamboo": {

    // Bamboo's HTTP 地址可以通过Marathon来访问
    // Marathon HTTP回调所使用；必须可以被Marathon访问
    "Host": "http://localhost:8000",

    // Proxy setting information is stored in Zookeeper
    // Bamboo will create this path if it does not already exist
    "Zookeeper": {
      // Use the same ZK setting if you run on the same ZK cluster
      "Host": "zk01.example.com:2812,zk02.example.com:2812",
      "Path": "/marathon-haproxy/state",
      "ReportingDelay": 5
        }
    }


    // Make sure using absolute path on production
    "HAProxy": {
        "TemplatePath": "/var/bamboo/haproxy_template.cfg",
        "OutputPath": "/etc/haproxy/haproxy.cfg",
        "ReloadCommand": "haproxy -f /etc/haproxy/haproxy.cfg -p         /var/run/haproxy.pid -D -sf $(cat /var/run/haproxy.pid)"
    },

    // Enable or disable StatsD event tracking
    "StatsD": {
        "Enabled": false,
        // StatsD or Graphite server host
        "Host": "localhost:8125",
        // StatsD namespace prefix
        // If you have multiple Bamboo instances, you might want to label each node
        // by bamboo-server.production.n1.
        "Prefix": "bamboo-server.production."
        }
    }
**定制带有Marathon App环境变量的HAProxy模板**  

Marathon app 环境变量可以在模板中被调用。随bamboo一起的默认模板知晓BAMBOO_TCP_PORT.当该变量被指定为Marathon app创建,该应用将被创建为TCP模式。例如：

    {
    "id": "FileServer",
    "cmd": "python -m SimpleHTTPServer $PORT0",
    "cpus": 0.1,
    "mem": 90,
    "ports": [0],
    "instances": 2,
    "env": {
        "BAMBOO_TCP_PORT": "1080",
        "MY_CUSTOM_ENV": "hello"
        }
    }


在该示例中，BMABOO_TCP_PORT和MY_CUSTOM_ENV可以在HAProxy模板中被访问到。其允许基于你的喜好实现弹性模板定制。  

**环境变量**  
production.json文件中的配置可以通过以下环境变量来重写。其通常在你为Bamboo和HAProxy构建一个Docker镜像的时候 会非常有帮助。如果这些未被定制那么配置文件中的默认值将会被使用。 


            环境变量                    对应参数   
            MARATHON_ENDPOINT           Marathon.Endpoint    
            BAMBOO_ENDPOINT             Marathon.Endpoint 
            BAMBOO_ZK_HOST              Bamboo.Zookeeper.Host  
            BAMBOO_ZK_PATH              Bamboo.Zookeeper.Path  
            HAPROXY_TEMPLATE_PATH       HAProxy.TemplatePath  
            HAPROXY_OUTPUT_PATH         HAProxy.OutputPath  
            HAPROXY_RELOAD_CMD          HAProxy.ReloadCommand  
            BAMBOO_DOCKER_AUTO_HOST     当 Bamboo 容器启动时， 设定 BAMBOO_ENDPOINT=$HOST 可以为任意值   
            STATSD_ENABLED              StatsD.Enabled  
            STATSD_PREFIX               StatsD.Prefix  
            STATSD_HOST                 StatsD.Host   


**REST APIs**

**GET /api/state**  
显示渲染模板使用的数据结构 

    curl -i http://localhost:8000/api/state

**GET /api/services**

显示所有的服务配置

    curl -i http://localhost:8000/api/services
    
示例结果:

    {
        "/authentication-service": {
            "Id": "/authentication-service",
            "Acl": "path_beg -i /authentication-service"
        },
        "/payment-service": {
            "Id": "/payment-service",
            "Acl": "path_beg -i /payment-service"
        }
    }
    
**POST /api/services**

为一个 Marathon 应用 ID 创建一个服务配置

    curl -i -X POST -d '{"id":"/ExampleAppGroup/app1","acl":"hdr(host) -i app-1.example.com"}' http://localhost:8000/api/services
    
**PUT /api/services/:id**

为一个 Marathon 应用升级一个现有的或创建一个新的服务配置。 :id 是该 Marathon 应用 ID

    curl -i -X PUT -d '{"id":"/ExampleAppGroup/app1", "acl":"path_beg -i /group/app-1"}' http://localhost:8000/api/services//ExampleAppGroup/app1  
    
注意： 创建语义将在版本 0.2.11中可用


**DELETE /api/services/:id**  
删除一个已经存在的服务配置. :id 为 Marathon 应用 ID  

       curl -i -X DELETE http://localhost:8000/api/services//ExampleAppGroup/app1  
**GET /status**  
Bamboo web应用的健康监测点

    curl -i http://localhost:8000/status  
    
**发布**    
我们推荐通过 deb 或 rpm 包来安装应用。代码仓库中包含了 [一个 Jenkins 构建脚本](https://github.com/QubitProducts/bamboo/blob/master/builder/ci-jenkins.sh) 和 [一个 deb 包构建脚本](https://github.com/QubitProducts/bamboo/blob/master/builder/build.sh)的示。可以通过阅读脚本中的注释来定制你的构建分发工作流。


更简单的版本，[安装 fpm](https://github.com/jordansissel/fpm) 并运行下面的命令：    
        go build bamboo.go  
        ./builder/build.sh  
一个 deb 包将在 ./builder 目录内被产生。你可以将其拷贝到一个服务器或发布到你自有的 apt 仓库。

deb包部署实例：  

* upstart 作业 [bamboo-server](https://github.com/QubitProducts/bamboo/blob/master/builder/bamboo-server)，比如。upstart 假定 /var/bamboo/production.json 被正确地配置。  
* 应用目录在 /opt/bamboo/ 之下  
* 配置和日志在 /var/bamboo/     
* 日志文件自动回滚  

在你不是用upstart的例子中，一个模板 init.d 服务将在 [init.d-bamboo-server](https://github.com/QubitProducts/bamboo/blob/master/builder/init.d-bamboo-server) 中被提供。通过以下方式来安装  

    sudo cp builder/init.d-bamboo-server /etc/init.d/bamboo-server
    sudo chown root:root /etc/init.d/bamboo-server
    sudo chmod 755 /etc/init.d/bamboo-server
    sudo update-rc.d "bamboo-server" defaults
    
你可以接着通过 sudo service bamboo-server start 来启动该服务器。其他的命令为： status, restart, stop  


**作为一个 Docker 容器**

这里有个 Dockerfile 其将允许 Bamboo 从一个 Docker 容器内部构建和运行。

**构建镜像**  


该 Docker 镜像可以在项目根目录中使用以下命令来构建并添加到你的本地仓库：   
        docker build -t bamboo .  
    
像 Docker 容器一样运行 Bamboo

当该镜像创建完成后，像一个容器一样运行就非常简单了 - 你仍然需要针对该镜像 提供配置作为环境变量。 对此  Docker 允许使用两个选项 - 使用 -e 选项或通过将配置写入一个文件并使用 --env-file 选项。 例如我们将使用前者并将端口 8000 和 80 映射至 docker 主机(很显然这个主机的配置将能够从该容器内被读取到)：  

    docker run -t -i --rm -p 8000:8000 -p 80:80 \
        -e MARATHON_ENDPOINT=http://marathon:8080 \
        -e BAMBOO_ENDPOINT=http://bamboo:8000 \
        -e BAMBOO_ZK_HOST=zk:2181 \
        -e BAMBOO_ZK_PATH=/bamboo \
        -e BIND=":8000"
        -e CONFIG_PATH="config/production.example.json"
        -e BAMBOO_DOCKER_AUTO_HOST=true
        bamboo

Bamboo可以在 Docker 镜像内部通过 supervisord 启动。[默认的 Supervisord 配置][1]重定向 stderr/stdout 日志到终端。如果你期望在生产环境中关闭调试信息，你可以使用一个[替代的配置][2].    
**开发和分发**

我们使用 [godep][3] 来管理 Go 包依赖；Goconvey 来做单元测试； CommonJS 和 SASS 为前端开发和构建分发。
  
* Golang 1.3
* Node.js 0.10.x+



**Golang:**  

    # 包管理器
    go get github.com/tools/godep
    # 测试工具集
    go get -t github.com/smartystreets/goconvey

    cd $GOPATH/src/github.com/QubitProducts/bamboo

    # 构建自己的应用
    go build

    # 运行测试
    goconvey
      
**Node.js UI 依赖:**

## 全局  
    npm install -g grunt-cli napa browserify node-static foreman karma-cli    
## 本地  
    npm install && napa  

## 启用一个与 Procfile 一起配置的 foreman 来构建 SASS 和 JavaScript   
    nf start


  [1]: https://github.com/QubitProducts/bamboo/blob/master/builder/supervisord.conf
  [2]: https://github.com/QubitProducts/bamboo/blob/master/builder/supervisord.conf.prod
  [3]: https://github.com/tools/godep
