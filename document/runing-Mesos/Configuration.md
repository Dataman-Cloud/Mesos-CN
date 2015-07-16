## mesos 配置向导
标签： 配置

mesos master 和 slave 可以通过命令行参数或环境变量来传递一系列的配置选项。通过运行 mesos-master –help 或者 mesos-slave –help 可以查看可用选项。每个选项可以通过两种方式设置：

 - 执行命令的时候使用 –-option_name=value 来传递配置选项。value 既可以是数值，也可以指定包含参数的文件 (--opthon_name=file://文件路径)。 该路径既可以是绝对路径，也可以是相对当前工作目录的相对路径。
 - 通过设定环境变量 MESOS_OPTION_NAME (变量名都以 MESOS_ 开头)

执行时会先读取环境变量，然后才看命令行参数
####重要的配置
如果你需要特定的需求，当配置 Mesos 的时候请参考 ./configure --help。另外，本文档列举了最新的选项快照。如果你想知道手头的版本支持哪些标记，你可以运行带有 --help 的命令，例如 mesos-master --help。

### Master 和 Slave 的配置选项

下列选项 master 和 slave 都支持：

                 标记                                  解释
        --external_log_file=VALUE  Specified the externally managed log file. This file will be exposed in the webui and HTTP api. This is useful when using stderr logging as the log file is otherwise unknown to Mesos.
 
        --firewall_rules=VALUE     该值是终端防火墙的规则（rules），可以为JSON类型的 rules 或包含 JSON
                                   类型 rules 的文件。文件路径可以为 
                                   file:///path/to/file 或者 /path/to/file。
                                   规则的格式请参考文件 flags.proto 中的防火墙信息。
                                   例如：
                                       {
                                         "disabled_endpoints" : {
                                            "paths" : [
                                               "/files/browse.json",
                                               "/slave(0)/stats.json",
                                            ]
                                        }
                                    }

        --[no-]help                输出帮助信息 (默认:否)
        --[no-]initialize_driver_logging	是否初始化 scheduler 和 executor driver 的 Google 
                                           loggin 机制（默认：是）
        --ip=VALUE                 监听的 IP 地址
        --log_dir=VALUE            输出日志文件的位置（无默认值，不指定就不生成日志文件。这个参数不影响输
                                   出到 stderr 的日志）
        --logbufsecs=VALUE         缓冲日志的时长（秒数）默认：0秒
        --loggin_level=VALUE       输出日志的起始级别，包括 'INFO', 'WARNING', 'ERROR'。如果使用了
                                   quiet 标记，只会影响到输出到 log_dir 的日志的级别（默认：INFO）
        --port=VALUE               监听端口（master默认5050，slave默认5051）
        --[no-]quiet               禁用输出日志到 sterr （默认:否）
        --[no-]version             显示版本并且退出 （默认：否）
 

##Master配置选项

*必选标记*

              标记                                    解释   
        --quorum=VALUE              The size of the quorum of replicas when using 'replicated_log' based registry. It is imperative to set this value to be a majority of masters i.e., quorum > (number of masters)/2.NOTE Not required if master is run in standalone mode (non-HA)。
        --work_dir=VALUE            注册表中的持久性信息的存放路径
        --zk=VALUE                  zookeeper URL（用于选举 master 的 leader）为以下之一：
                                    zk://host1:port1,host2:port2,.../path
                                    zk://username:password@host1:port1,host2:port2,.../path
                                    file:///path/to/file
                                    注意：如果 master 在单机模式下运行就不需要该设置(non-HA)。
*可选标记*

              标记                                    解释 
        --acls=VALUE               JSON 格式的 ACL，请记住你也可以使用 file:///path/to/file 或者 /path/to/file 指定包含该列表的文件，格式请参考文件 mesos.proto 中的 ACLs protobuf 段落
                                   JSON文件例子：
                                   {
                                        "register_frameworks": [
                                        {
                                    "principals": { "type": "ANY" },
                                    "roles": { "values": ["a"] }
                                    }
                                        ],
                                    "run_tasks": [
                                        {
                                    "principals": { "values": ["a", "b"] },
                                    "users": { "values": ["c"] }
                                        }
                                    ],
                                "shutdown_frameworks": [
                                    {
                                    "principals": { "values": ["a", "b"] },
                                    "framework_principals": {  "values":                           ["c"]}
                                            }
                                        ]
                                    }
    --allocation_interval=VALUE 性能(批量)分配之间多长时间用来等待(比如，500ms,1sec等).(默认值：1secs) 
    --allocator=VALUE           针对框架做资源分配的分配器。使用默认的HierarchicalDRF分配器或通过使用--modules来加载备用的分配器模块。(默认：HierarchicalDRF)
    --[no-]authenticate         如果身份验证设置为'true'，只有经过身份验证的框架才被允许被注册。如果设置为'false'未经身份验证的框架也能允许被注册。(默认：false)
    --[no-]authenticate_slaves  如果设置为'true' 通过身份验证的slave才能被注册，如果设置为'false'未经身份验证的slaves也能被注册。(默认：false)
    --authenticators=VALUE      验证器被实现用于当框架或slave进行身份验证时，可以使用默认的crammd5或者通过使用--modules来加载备用的验证器模块。(默认:crammd5)
    --cluster=VALUE             显示在webui上的可被人读取的集群名称。
    --credentials=VALUE         一个包含一系列凭据列表的文件的路径，每行包含由空格分割的'principal' 和 'secret'，或是包含凭证的JSON格式文件的路径，路径格式必须为file:///文件路径或/文件路径之一。
                                json文件样例：
                                {
                                    "credentials": [
                                    {
                                        "principal": "sherman",
                                        "secret": "kitesurf"
                                    }
                                        ]
                                }
                                文本文件样例：
                                username secret
    --external_log_file=VALUE    指定外部的管理日志文件。该文件将暴露在webui和HTTP api中，当用stderr记录日志时很有用，否则mesos日志文件是未知的。

--framework_sorter=VALUE 给定用户的工作框架之间资源分配的策略。选项和user_allocator相同。默认：drf。

--hooks=VALUE 钩模块以逗号分割的列表安装在住节点内。

--hostname=VALUE 主节点通知zookeeper里面的主机名。如果没有设置将从绑定的IP地址解析。

--[no-]log_auto_initialize 是否自动初始化注册日志，如果这是为false，必须在第一次使用时手动初始化。默认：true。

--modules 加载模块列表并用于内部的子系统。使用—modules=filepath指定包含JSON格式文件的模块列表。可以使用file:///path/to/file 或者/path/to/file指定JSON格式的文件。用—modules=”{...}”指定模块列表。

--offer_timeout=VALUE 从工作框架提议撤销之前的持续时间。有助于当工作框架坚持提议，或者不小心丢了提议时的公平。

--rate_limits=VALUE 值可以是一个JSON格式的字符串或者是一个文件路径包含JSON格式的文件用户框架的速率限制。文件路径可以是file:///path/to/file或者/path/to/file。

--recovery_slave_removal_limit=VALUE 对于故障转移，限制百分比从节点冲注册表中删除和关机后重新登记超时时间间隔。如果超出限制主节点将删除从节点。这可以用来为生产环境提供安全保障。

生产环境希望master故障切换，顶多一定比例的从节点会永久失败例如由于机架失败。设定此限制将确保需要人介入从节点意外普遍故障发生在集群中。值[0%-100%]。默认:100%.

--slave_removal_rate_limit=VALUE 最大速率在从节点被删除从主节点健康检查失败时。默认情况下从节点将被删除只要健康检查失败。值的形式是‘从节点数量’/‘持续时间’。

--registry=VALUE 注册表持久性策略；可用选项是 'replicated_log', 'in_memory'，进行测试。默认：replicated_log。

--registry_fetch_timeout=VALUE 持续时间等待从注册表获取数据后其中操作被认为失败。默认：1mins

--registry_store_timeout=VALUE 持续时间等待将数据存储到注册表视为失败。默认：5secs

--[no-]registry_strict master是否将会采取行动持续信息基础存储到注册表中。

--roles=VALUE 逗号分割的分配角色列表，在这个集群可能属于工作框架。

--[no-]root_submissions 可以root提交框架吗。默认：true

--slave_reregister_timeout=VALUE 超时时间内所有的从节点预计将重新注册时，选举出一个新的master。如果超时没有重新注册上从节点将从注册表里面删除并关闭如果他们试图与主服务器通信。

--user_sorter=VALUE 用户之间资源分配策略，可能是dominant_resource_fairness (drf)。默认：drf

--webui_dir=VALUE 网页文件目录，默认：/usr/local/share/mesos/webui

--weights=VALUE 逗号分割的role/weight列表，成对表单'role=weight,role=weight'。 weights是用来表示形式的优先级。

--whitelist=VALUE 一个文件名包含从节点作为通知的列表。这个文件是个监测，并且定期重新读取刷新从节点名单。默认：None。

--zk_session_timeout=VALUE zookeeper session超时。



从节点选项

必须项：

--master=VALUE 从节点链接到主节点的地址，有三种方式1.--master=masterip1:port,masterip2:port 2.--master=zk://host1:port1,host2:port2,.../path --master=zk://username:password@host1:port1,host2:port2,.../path 3. file:///path/to/file



可选项

--attributes=VALUE 机器的属性：rack:2或者rack:2;u:1

--authenticatee=VALUE 用于主节点身份验证，默认crammd5，或者用—modules加载备用模块。默认：crammd5。

--[no-]cgroups_enable_cfs 通过限制CFS带宽来限制CPU资源. 默认: defult

--cgroups_hierarchy=VALUE cgroups的路径根位置. 默认: /sys/fs/cgroup

--[no-]cgroups_limit_swap 用于内存和swap的限制, 默认: false, 只限制内存

--cgroups_root=VALUE 根cgroup的命名. 默认: mesos

--container_disk_watch_interval=VALUE 用于查询容器中磁盘配额时间间隔. 被用于posix/disk的时间间隔, 默认: 15秒

--containerizer_path=VALUE 当外部隔离机制被激活时(--isolation=external), 外部容器被执行的路径 

--containerizers=VALUE 用逗号把一组容器隔开, 以达到对容器的实现. 包括mesos, external, docker在Linux中. 默认: mesos

--credential=VALUE 一行包含"principal"和"secret"由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是file://path/to/file. 也可以使用路径file:///path/to/file, 从文件中读取值 。
JSON 文件例子:
```
{
  "principal": "username",
  "secret": "secret"
}
```
--default_container_image=VALUE   当使用 external containerizer 时，如果没有在一个 task 上指定 ，则使用 default container image 。

--default_container_info=VALUE  JSON格式的 CONTAINERINFO 将包含到任何没有指定 ContainerInfo 的 ExecutorInfo 中。

例如：
```
{
  "type": "MESOS",
  "volumes": [
    {
      "host_path": "./.private/tmp",
      "container_path": "/tmp",
      "mode": "RW"
    }
  ]
}
```

--default_role=VALUE 任何用 --resources 忽略　role 的资源，以及在没有　--resources 标记，但被检测到的资源。都将使用默认的这个值。

--disk_watch_interval=VALUE 周期性(例如 10 S ,2 MIN 等)检查硬盘的使用情况。默认是 1 MIN 。

--docker=VALUE docker 容器化的可执行文件的绝对路径。( 默认: docker )

--docker_remove_delay=VALUE 移除 docker 前等待的时间 （ 如 3 天，2 周 等）。默认为 6 小时。

--[no-]docker_kill_orphans 允许 docker kill 掉 orphaned containers 。当你启动多个 slave 在相同的 OS ，你应该考虑将此值设为 false 。 然而，你还应该确保启用　checkpoint　，这样可以重用相同 id 的 slave 。否则当 slave 重启后，docker 任务不会被清除掉。( 默认为　true ) 。

--docker_sock=VALUE UNIX socket 路径被挂载到 docker executor 以提供 docker CLI 
有权访问 docker daemon 。它必须是用于使用 slave 的 docker 镜像的路径 （ 默认： /var/run/docker.sock ）。

--docker_mesos_image=VALUE docker 镜像用于启动这个 mesos 实例。如果一个镜像被指定，docker containerizer 假定 slave 运行在 docker 容器中，并当 slave 重启和恢复时启动 executor 恢复他们。

--docker_sandbox_directory=VALUE 描述沙盒在容器中被映射到的绝对路径。（ 默认：  /mnt/mesos/sandbox ）。

--docker_stop_timeout=VALUE 杀死实例后，停止它的间隔时间 （ 默认： 0 Secs ）。

-[no-]enforce_container_disk_quota 是否为容器启用磁盘定额。这个标记用来 ' posix/disk ' 隔离。 （ 默认: false ）。

--executor_environment_variables JSON 对象，代表传递到 executor 的环境变量， and thus subsequently task(s)  。默认情况下 executor 将继承 slave 的环境变量。例如：
```
{
  "PATH": "/bin:/usr/bin",
  "LD_LIBRARY_PATH": "/usr/local/lib"
}
```

--executor_registration_timeout=VALUE  挂起或者关闭前，等待 executor 注册 slave 的时间。（ 例如，60 S，3 mins 等 ）。默认为 1 MIN 。

--executor_shutdown_grace_period=VALUE 等待 executor 关闭的时间。( 例如, 60 S, 3 mins 等 )。默认为 5 S 。

--frameworks_home=VALUE 相对于 executor 的路径前缀的 URI 。

--gc_delay=VALUE 清理 executor 目录的延迟时间（ 例如，3 天 或 2 周 等 ）。注意，根据实际可用磁盘的情况，这个值可能会小些（ 默认：1 周 ）。

--gc_disk_headroom = VALUE 用于调整 executor 目录的最大磁盘空间。计算方法为 gc_delay * max(0.0, (1.0 - gc_disk_headroom - disk usage)) 每个 --disk_watch_interval 期间，gc_disk_headroom 都必须在 0.0 到 1.0 之间。（ 默认：0.1 ）。

--hadoop_home=VALUE Hadoop 的安装路径。

--hooks=VALUE A comma separated list of hook modules to be installed inside master.

--hostname=VALUE slave 的 hostname 。

--isolation = VALUE 隔离机制的使用。 例如  ' posix/cpu,posix/mem ', ' cgroups/cpu,cgroups/mem '  或者 network/port_mapping 及 'external' 或者使用 ***--modules*** 标记代替隔离模块。注意，这个标记只用于 Mesos Containerizer （ 默认：posix/cpu, posix/mem ）。

--launcher_dir=VALUE Mesos 二进制目录路径 （ 默认： /usr/local/lib/mesos ）。

--modules=VALUE 待加载的模块列表，并提供给内部的子系统。你也可以使用 file:///path/to/file 或者 /path/to/file 参数值格式从一个文件中读取值。使用 ***--modules="{...}"*** 指定模块内嵌的列表。
JSON 文件例子：
```
{
  "libraries": [
    {
      "file": "/path/to/libfoo.so",
      "modules": [
        {
          "name": "org_apache_mesos_bar",
          "parameters": [
            {
              "key": "X",
              "value": "Y"
            }
          ]
        },
        {
          "name": "org_apache_mesos_baz"
        }
      ]
    },
    {
      "name": "qux",
      "modules": [
        {
          "name": "org_apache_mesos_norf"
        }
      ]
    }
  ]
}
```

--oversubscribed_resources_interval=VALUE Slave 会定期向 master 更新自己有效的，可以分配的资源。（ 默认：15 S ）。

 
--perf_duration=VALUE Duration of a perf stat sample 。持续时间必须比 perf_interval 少。默认为 10 S。

--perf_events=VALUE 当使用 perf_event  隔离的时候，List of command-separated perf events to sample for each container 。默认为 None。运行 ' perf list ' 命令查看所有事件。当在 PerfStatistics protobuf 中 reported 的时候，Event names are sanitized by downcasing 以及使用下划线代替连字符。 例如，cpu-cycles 变为 cpu_cycles。在 PerfStatistics protobuf 中可以看到所有名字。

--perf_interval = VALUE Interval between the start of perf stat samples. Perf samples are obtained periodically according to perf_interval and the most recently obtained sample is returned rather than sampling on demand。
基于此原因，perf_interval 是独立的监视资源时间间隔的。（ 默认： 1 mins ）。

--qos_controller=VALUE The name of the QoS Controller to use for oversubscription.

--qos_correction_interval_min=VALUE The slave polls and carries out QoS corrections from the QoS Controller based on its observed performance of running tasks. 这些校正之间的最小间隔有此标记指定。 （ 默认： 0 secs ）。

--recover = VALUE 是否要恢复状态更新以及重新连接 old executor 。" recover " 是有效值。reconnect : 重新连接任何活着的 old executor。 cleanup : 杀死并退出所有活着的 old exetuor 。当你要做一个不兼容的 slave　活着 executor 升级时，可以用这个选项。注意：当 slave 没有设置 checkpointed 时，recovery 不能执行，并且  slave 作为一个新的 slave 注册到 master。 ( 默认： reconnect )

--recovery_timeout=VALUE 分配给 slave 恢复的时间。如果　slave 恢复所用的时间超过 recovery_timeout，将会被终止。( 默认：15 min )

--registration_backoff_factor=VALUE slave 最初挑选一个随机的时间段[0,B],其中 b = registration_backoff_factor ,( 重新注册 )注册到一个新的 master。 后续重试都基于此时间段成倍扩大 ，最多为 1 mins。( 默认: 1 secs )

--resource_estimator=VALUE 用于 oversubscription 的 resource estimator 的名称。

--resource_monitoring_interval=VALUE executor 资源使用监视周期。（ 例如，10 secs , 1 min 等 ）.( 默认： 1secs )。

--resources=VALUE 每个 slave 总的可消耗资源。格式：***name(role):value;name(role):value...***。

--[no-]revocable_cpu_low_priority 通过 revocable CPU 以相对低的优先级运行 containers 。 目前只支持 cgroups/cpu isolator 。( 默认: true )

--slave_subsystems=VALUE List of comma-separated cgroup subsystems to run the slave binary in 。例如，***memory*** , ***cpuacct*** 。默认为 none 。此功能用于资源的监视以及 no cgroup 下限制设置，它们从 root mesos cgroup 继承而来。

--[no-]strict 如果 strict = true ，任何以及所有错误恢复都被认为是致命的。反之，恢复期间，任何预期的错误都会被忽略。 ( 默认： true )

--[no-]switch_user Whether to run tasks as the user who submitted them ，而不是运行 slave 的 user ( 需要 setuid 权限)。 ( 默认: true )

--fetcher_cache_size=VALUE 以字节为单位的 fetcher cache 大小。( 默认: 2 GB )

--fetcher_cache_dir=VALUE fetcher cache 的父目录。默认在工作目录中，所以一切都可以在测试环境中保存或删除。然而，典型的生产方案使用的是单独的 cache volume 。首先，它不意味着要备份。其次，要避免沙盒目录和缓存目录以不可预知的方式通过共享空间相互干扰。因此建议，明确设置缓存目录。 ( 默认: /tmp/mesos/fetch )

--work_dir=VALUE framework 工作目录的路径。( 默认: /tmp/mesos )

#####当配置了 ' –with-network-isolator ', 以下命令才会生效：

--ephemeral_ports_per_container=VALUE 有网络隔离器分配临时端口给一个容器。此端口号必须是 2 的倍数。( 默认: 1024 )

--eth0_name = VALUE 公网接口的名称 ( 如 eth0 )。如果没有指定，网络隔离器会尝试基于主机的默认网关来猜测它。

--lo_name=VALUE 网络 loopback 接口的名称( 例如, lo )。如果没有指定，网络隔离器会尝试猜测它。

--egress_rate_limit_per_container=VALUE 每个容器的出口流量限制，单位是 字节/每秒。如果没有指定或指定为零，网络隔离器不会强制限制容器的出口流量。这个标记使用字节类型, 定义在 stout 。

--[no-]network_enable_socket_statistics_summary 是否从每个容器收集 socket 统计摘要。这个标记被用在  'network/port_mapping' 隔离器。 ( 默认: false )

--[no-]network_enable_socket_statistics_details 是否从每个容器收集 socket 细节信息。该标记用于 ' network/port_mapping ' 隔离器。( 默认: false )


###Mesos构建配置选项

####配置脚本有以下可选的功能标志

--enable-shared[=PKGS] 建立共享库 [ 默认: yes ]

--enable-static[=PKGS] 建立静态库 [ 默认: yes ]

--enable-fast-install[=PKGS] 快速安装 [ 默认: yes ]

--disable-libtool-lock 避免锁死 ( 可能会破坏并发的编译 )

--disable-java don't build Java bindings

--disable-python don't build Python bindings

--enable-debug 启用 Debug 调试。如果设置了 CFLAGS/CXXFLAGS ，这个选项不会改变它们。默认: no 

--enable-optimize 启用优化 如果设置了 CFLAGS/CXXFLAGS ，这个选项不会改变它们。默认: no 

--disable-bundled 预装相关依赖，而不是绑定 libraries 。

--disable-bundled-distribute excludes building and using the bundled distribute package in lieu of an installed version in PYTHONPATH

--disable-bundled-pip	excludes building and using the bundled pip package in lieu of an installed version in PYTHONPATH

--disable-bundled-wheel	excludes building and using the bundled wheel package in lieu of an installed version in PYTHONPATH

--disable-python-dependency-install 在 make install 期间 python packages 已经安装时，没有外部依赖正在下载或者安装。

####配置脚本有以下可选的 packages 选项：

--with-gnu-ld 假设 C 编译器使用 GNU ld [ 默认: no ]

--with-sysroot=DIR 在 DIR 中搜索依赖库 ( 或者如果编译器的 sysroot 没有指定 )

--with-zookeeper[=DIR]	excludes building and using the bundled ZooKeeper package in lieu of an installed version at a location prefixed by the given path

--with-leveldb[=DIR]	excludes building and using the bundled LevelDB package in lieu of an installed version at a location prefixed by the given path

--with-glog[=DIR]	excludes building and using the bundled glog package in lieu of an installed version at a location prefixed by the given path

--with-protobuf[=DIR]	excludes building and using the bundled protobuf package in lieu of an installed version at a location prefixed by the given path

--with-gmock[=DIR]	excludes building and using the bundled gmock package in lieu of an installed version at a location prefixed by the given path

--with-curl=[=DIR] 指定在哪里找到 curl library

--with-sasl=[=DIR] 指定在哪里找到 sasl2 library

--with-zlib=[=DIR] 指定在哪里找到 zlib library

--with-apr=[=DIR] 指定在哪里找到 apr-1 library

--with-svn=[=DIR] 指定在哪里找到 svn-1 library

--with-network-isolator 建立网络隔离

####一些 influential 的环境变量配置脚本：

Use these variables to override the choices made by `configure' or to help it to find libraries and programs with nonstandard names/locations.

JAVA_HOME  JDK 根目录

JAVA_CPPFLAGS  JNI 目录

JAVA_JVM_LIBRARY libjvm.so 的完整路径

MAVEN_HOME  mvn 的根目录

PROTOBUF_JAR　在 prefixed builds　上完整的 protobuf jar　路径

PYTHON　指定使用哪个 Python 解析器

PYTHON_VERSION 已经安装并使用的 Python 版本号 例如，'2.3'。该字符串将追加到 Python 解析器

