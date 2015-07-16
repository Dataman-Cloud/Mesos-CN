#Marathon Command Line Flags
###这些命令行标记控制着Marathon的核心功能。

注-可以使用环境变量设定命令行标记

##核心功能
标记可以通过环境变量设置 MARATHON_OPTION_NAME(marathon 的变量名称以作为MARATHON_开头)，例如 标记—master可以写成 MARATHON_MASTER。请注意命令行标记优先级高于环境变量。如果设置了环境变量同时也有相同的命令行标记，那么环境变量将被忽略。

##必需标记


- `--master` (必须) ：mesos master的URL。格式是用逗号分隔的多个主机地址，例如 `zk://host1:port,host2:port/mesos`。特别注意，如果使用了zookeeper，主机地址将以`zk://`开头，以/mesos结尾。如果没有使用zookeeper，可以使用标准URL，例如 `http://localhost`

##可选标记


- `--artifact_store`(默认值：None)：工件存储的URL。例如：”`hdfs://localhost:54310/path/to/store`”, “`file:///var/log/store`”。



- --access_control_allow_origin (默认值：None) 允许以指定的域名发起 HTTP 请求，格式以逗号分隔，不设置则为默认值。例如：“*”，或者 “http://localhost:8888, http://domain.com”。

--checkpoint (默认值：true) 开启任务检查，需要在从节点上开启检查。允许任务在mesos-slave重启和marathon任务调度故障切换期间继续运行。参见—failover_timeout的描述

--executor (默认值： “//cmd”) 指定执行器。（待审核）

--failover_timeout (默认值：604800秒（1周）) 为mesos 设置故障恢复超时。如果一个新的marathon实例在故障恢复中长时间未向mesos注册，到达超时时间后，mesos关闭marathon启动的所有运行中任务；

--framework_name (默认值：marathon-VERSION) ：marathon注册在mesos上的名字。

--ha (默认值：true) ：在HA模式以选举方式运行marathon。允许再启动多个marathon,并限制只能在HA模式下启动。这个模式需要运行zookeeper。

--hostname (默认值：主机的hostname) ：主机名存储在zookeeper，其他的备用主机可以通过选举成leader。注：默认值由 InetAddress.getLocalHost 确定；

--webui_url (默认值：None)：marathon网页的URL。用这个链接通过mesos返回到marathon页面。如果没有设置，选举出的实例的url将发送给mesos。

--local_port_max (默认值：20000)：分配给程序的最大端口号。

--local_port_min (默认值：10000)：分配给程序的最小端口号。

--mesos_role (默认值：None) ： 针对该 marathon 的mesos职责（角色？）。

--mesos_user (默认值：当前用户)： 该 marathon框架的mesos用户。

--reconciliation_interval_delay (默认值：15000（15秒）)：毫秒单位。当延迟到达，marathon 开始周期性地执行任务协调操作；

--reconciliation_interval (默认值：300000（5分钟）)：毫秒单位。任务协调操作的周期；

--scale_apps_initial_delay (默认值：150000（15秒）)：毫秒单位。当延迟到达，marathon 开始周期性地执行应用扩展操作；

--scale_apps_interval (默认值：300000（5分钟）)：毫秒单位。应用扩展操作的周期；

--task_launch_timeout （默认值：300000（5分钟））：已过期 ，单位毫秒。如果一个任务在此时间内没有进入TASKRUNNING状态就杀掉这个任务。注：这是MESOS-1922的临时解决方案，在以后的版本中会删掉此项。

--event_subscriber （默认值：None）：启动事件订阅模块。目前唯一的有效值是http_callback。

--http_endpoints （默认值：None）：预设的http callback url。只在与—event_subscriber http_callback 配合使用时有效。额外的 callback url 可以通过 REST API 动态设置。

--zk (默认值：None)：zookeeper用于存储状态的url。格式：zk://host1:port1,host2:port2,.../path

--zk_max_versions (默认值：None)：一个 entity 存储的版本数。

--zk_timeout (默认值：10000（10秒）) zookeeper超时时间

—mesos_authentication_principal：用于mesos身份验证。

--mesos_authentication_secret_file：包含认证密钥的 mesos 密钥文件的路径；

--marathon_store_timeout (默认值：2000（2秒）)：毫秒单位，持久操作的超时时间。


Web站点标记
web站点标记控制着marathon的web站点行为，包括面向用户的站点和REST API。他们继承自 Chaos 库，marathon 和伙伴项目 chronos 都基于该库。

可选标记
--assets_path (默认值：None)：web UI 可以从本地文件系统加载资源，如果没有指定，将从JAR包里面加载资源。

--http_address (默认值：all)：监听http请求的地址。

--http_credentials (默认值：None)：用于访问http服务的凭证， 格式 username:password。用户名不能包含(:)。也可以通过环境变量MESOSPHERE_HTTP_CREDENTIALS设置。

--http_port (默认值：8080):监听http请求的端口。

—disable_http：完全禁用http。设置前提条件：配置了https访问。https保持启用。

--https_address (默认值：all)：监听https请求的地址。

--https_port (默认值：8443)：监听https请求的端口。设置前提条件：—ssl_keystore_path和—ssl_keystore_password已经设置。

--http_realm (默认值：Mesosphere)：安全范围（对应 aka 中的“area”），与凭证协作使用；

--ssl_keystore_path (默认值：None)：SSL密钥存储的路径，如果提供次选项，将会HTTPS（SSL）。需要—ssl_keystore_password。可以通过设置环境变量MESOSPHERE_KEYSTORE_PATH设置。

--ssl_keystore_password (默认值：None)：密钥存储库密码。如果设置了—ssl_keystore_path，则该项需要被设置。可以通过设置环境变量MESOSPHERE_KEYSTORE_PASS设置。