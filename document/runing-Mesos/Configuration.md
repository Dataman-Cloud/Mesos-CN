## mesos 配置向导

mesos master 和 slave 可以通过命令行参数或环境变量来传递一系列的配置选项。通过运行 mesos-master –help 或者 mesos-slave –help 可以查看相关的可用选项。每个选项可以通过以下两种方式设置：

 - 执行命令的时候使用 –-option_name=value 来传递配置选项。value 既可以是数值，也可以指定包含参数的文件 (--opthon_name=file://文件路径)。 该路径既可以是绝对路径，也可以是相对当前工作目录的相对路径。

 - 通过设定环境变量 MESOS_OPTION_NAME (变量名都以 MESOS_ 开头)  
执行时会先读取环境变量，然后才看命令行参数

####重要的配置
如果你有特定的需求，当配置 Mesos 的时候请参考 ./configure --help。另外，本文档列举了最新的配置选项。如果你想知道手头的版本支持哪些标志位，你可以运行带有 --help 的命令，例如 mesos-master --help。

### Master 和 Slave 的配置选项

下列选项 都被 master 和 slave 所支持：

                 标志位                                 解释
        --external_log_file=VALUE  Specified the externally managed log file. This file will be exposed in the webui and HTTP api. This is useful when using stderr logging as the log file is otherwise unknown to Mesos.
        --firewall_rules=VALUE     该值是终端防火墙的规则（rules），可以为JSON 类型的 rules 或包含 JSON
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
        --[no-]initialize_driver_logging	是否初始化 scheduler 和 executor driver 的
                                           logging 机制（默认：是）
        --ip=VALUE                 监听的 IP 地址
        --log_dir=VALUE            输出日志文件的位置（无默认值，不指定就不生成日志文件。这个参数不影响输
                                   出到 stderr 的日志）
        --logbufsecs=VALUE         缓冲日志的时长（秒数）默认：0秒
        --loggin_level=VALUE       输出日志的起始级别，包括 'INFO', 'WARNING', 'ERROR'。如果使用了
                                   quiet 标记，只会影响到输出到 log_dir 的日志的级别（默认：INFO）
        --port=VALUE               监听端口（master默认5050，slave默认5051）
        --[no-]quiet               禁用输出日志到 sterr （默认:否）
        --[no-]version             显示版本并退出 （默认：否）
 
       

##Master 配置选项

*必选标志位*

              标志位                                    解释   
        --quorum=VALUE              当使用 'replicated_log' 为基础的注册时副本仲裁数量的多少。必须将该值设定为大多数 masters 比如， quorum > ( masters 总量)/2。注意：如果 master 是运行于一个单实例模式则不需要设定（non-HA）。
        --work_dir=VALUE            注册表中的持久性信息的存放路径
        --zk=VALUE                  zookeeper URL（用于选举 master 的 leader）为以下之一：
                                    zk://host1:port1,host2:port2,.../path
                                    zk://username:password@host1:port1,host2:port2,.../path
                                    file:///path/to/file
                                    注意：如果 master 在单机模式下运行就不需要该设置(non-HA)。
*可选标志位*

              标志位                                    解释 
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
    --allocation_interval=VALUE 两次资源分配（批量）之间的等待时长 （500ms,1sec等）。（默认值：1secs）
    --allocator=VALUE           指定被框架（Framework）使用的资源分配器，默认使用 HierarchicalDRF 分配器。通过 --modules 参数可以加载其他类型的分配器模块。(默认：HierarchicalDRF)
    --[no-]authenticate         如果身份验证设置为 'true'，只允许经过身份验证的框架注册。如果设置为 'false'，未经身份验证的框架也允许被注册。(默认：false)
    --[no-]authenticate_slaves  如果设置为 'true'， 只有通过身份验证的 slave 才允许被注册；如果设置为 'false' 未经身份验证的 slave 也允许被注册。(默认：false)
    --authenticators=VALUE      框架或 slave 进行身份验证时使用的验证器，默认为 crammd5。也可以使用 --modules 来加载其他验证器模块。(默认:crammd5)
    --cluster=VALUE             具有可读性的集群名称，在 webui 上显示。
    --credentials=VALUE         包含 credential 列表的文件路径。文件格式即可以是普通文本，也可以是 JSON 格式。如果是普通文本，每行包含由空格分割的 principal 和  secret。文件路径格式必须为 file:///path/to/file 或 /path/to/file。
                                JSON 文件样例：
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
    --framework_sorter=VALUE    (default: drf) 用于一个给定用户的架构之间分配资源的策略。该选项和 user_allocator 选项相同。 (默认： drf)
    --hooks=VALUE               安装在 master 内的钩子模块（hook module）列表，名子以逗号进行分隔
    --hostname=VALUE            该 master 在 zookeeper 里登记的主机名。如果没有设置，将从绑定的IP地址进行解析。
    --[no-]log_auto_initialize  是否通过使用该注册来自动初始化复制的 log .如果被设定为 false ， log 将会在最初使用的时候被手动初始化。(默认： true)
    --max_slave_ping_timeouts=VALUE	 master 尝试 ping slave 的最高连续失败次数，超过这个限制的 slave 将被移除。（默认：5）
    --modules                   需要加载的模块列表，它们可以被内部的子系统调用。使用 —modules=filepath  指定包含模块列表的文件（JSON格式）。可以使用 file:///path/to/file 或者 /path/to/file 来指定文件，或者直接用 —modules=“{...}” 参数指定模块列表。
                                JSON 格式文件举例:
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
     --offer_timeout=VALUE    offer 从一个 framework 中被回收之前的时间间隔。 其保证了公平性当运行中的 frameworks 留住 offers, 或 frameworks 意外的丢弃 offers。
     --rate_limits=VALUE     该值可以为一个 JSON 格式的速率限制或一个文件路径包含了被 framework 限速 所使用的 JSON 格式的速率限制。请记住你也可以是使用 file:///path/to/file 或 /path/to/file 参数值格式来将该 JSON 写入至一个文件。
                             期望的格式请参考 mesos.proto 中的 RateLimits protobuf.
                             例子：
                             {
                               "limits": [
                                    {
                                      "principal": "foo",
                                      "qps": 55.5
                                    },
                                    {
                                      "principal": "bar"
                                    }
                                        ],
                                      "aggregate_default_qps": 33.3
                              }
     --recovery_slave_removal_limit=VALUE     针对故障转移，限制上的百分比的 slaves 可以从注册中移除并关机在重新注册的超时时间到了之后。 如果该限制被突破， master 将实行故障转移而不是移除 slaves.这可被用来针对生产环境提供安全保障。生产环境可能期望在 Master 故障转移过程中， 最多一定百分比的 slaves 将永久性的挂掉 (比如, 由于 rack-level 的故障)。设定该限制可以保证一个人需要参与进来当在该集群中一个非预期的大范围的 slave 故障发生。值: [0%-100%] (默认: 100%)
    --registry=VALUE          注册表持久化策略。可用选项有 'replicated_log','in_memory'（用于测试）。默认：replicated_log。
    --registry_fetch_timeout=VALUE 在操作被认为是一个失败后的为了从注册中提取数据的等待的时间间隔.(默认： 1mins)
    --registry_store_timeout=VALUE 等待的时间周期为了当操作被认为一个失败的时候将数据存储入注册机。 (默认：5secs)
    --[no-]registry_strict    无论 Master 是否将基于注册机中存储的持久信息来采取行动。设定改值为 false 意味着注册员将永远拒绝入列，出列和一个 slave 的移除。所以， 'false' 可以用来在一个运行的集群上来引导持久化的状态。注意： 该标志位是 *experimental* 而且还不能在应用中使用.(默认: false)
    --roles=VALUE             其 frameworks 在这个集群中可能归属于的用逗号分离的一系列指派的角色。
    --[no-]root_submissions   root 是否可以提交 frameworks? (默认: true)
    --slave_ping_timeout=VALUE	在每个 slave 被期望从一个 master 回应一个 ping 值的超时时间。 Slaves 如果不是在  `max_slave_ping_timeouts` 回复，ping 从新尝试将被移除.  (默认: 15secs)
    --slave_removal_rate_limit=VALUE	最大的比例(e.g., 1/10mins, 2/3hrs, etc) 对于那个 slaves 将被从 master 中移除当他们遇到健康检测失败。默认的是 slave 将尽可能快的被移除当它们遇到健康监测失败。值为 'Number of slaves'/'Duration' 的模式。
    --slave_reregister_timeout=VALUE	在所有的 slaves被期望重新注册当一个新的 master 被选举为 leader的超时时间。 Slaves 其不会在此超时时间内被重新注册将被从注册中移除并将被关掉如果它们尝试去与 master 通信。注意： 该值将被设置为最少 10mins. (默认: 10mins)
    --user_sorter=VALUE	     被用来在用户中分配资源的策略。可以为以下之一：dominant_resource_fairness (drf) (default: drf) 
    --webui_dir=VALUE        管理页面的网页文件的目录，默认：/usr/local/share/mesos/webui
    --weights=VALUE          逗号分割的角色/权重列表，成对表单 'role=weight,role=weight'。 weights是用来表达优先级。
    --whitelist=VALUE         一个 文件名器包含一系列的 slaves （每行一个）来通告 offers.该文件被观测，并周期性的重读取来刷新 slave 白名单。 默认的这里没有白名单/所有机器被接收. (默认: None)
                                例子：
                                file:///etc/mesos/slave_whitelist
    --zk_session_timeout=VALUE  zookeeper 的 session 超时时长。

*使用 ‘–with-network-isolator’ 配置时允许使用的参数*

    --max_executors_per_slave=VALUE	每个 Slave 上最大允许的执行器数量。网络监控和隔离机制强行限制每个执行器使用的端口资源，所以每个 slave 上智能跑一定数量的执行器。这个标志位是用来避免框架接收了某些资源 offer，执行的时候却发现该 slave 上端口已经被分配完毕。


## Slave 选项

*必选项*

    --master=VALUE       slave 连接到 master 的 URL，有三种连接方式：
                         1. master 的主机名或者 IP 地址，如果是多个地址可以用逗号隔开，例如：
                               --master=localhost:5050
                               --master=10.0.0.5:5050,10.0.0.6:5050
                         2. zookeeper 或仲裁 主机名/ip +端口  master 注册地址
                               --master=zk://host1:port1,host2:port2,.../path
                               --master=zk://username:password@host1:port1,host2:port2,.../path
                         一个文件对应的一个路径包含以上选项的任意一个。你也可以使用 file:///path/to/file 语句来从一个包含以上任意一个选项的文件中读取参数


*可选项*

    --attributes=VALUE    机器的属性：rack:2或者rack:2;u:1
    --authenticatee=VALUE 用于主节点身份验证，默认crammd5，或者用—modules加载备用模块。默认：crammd5。
    --[no-]cgroups_enable_cfs 通过限制CFS带宽来限制CPU资源. 默认: defult
    --cgroups_hierarchy=VALUE cgroups的根路径位置. 默认: /sys/fs/cgroup
    --[no-]cgroups_limit_swap 用于内存和swap的限制, 默认: false, 只限制内存
    --cgroups_root=VALUE      根cgroup的命名. 默认: mesos
    --container_disk_watch_interval=VALUE 用于查询容器中磁盘配额的时间间隔. 被用于posix/disk的时间间隔, 默认: 15秒
    --containerizer_path=VALUE 当外部隔离机制被激活时(--isolation=external), 外部容器被执行的路径 
    --containerizers=VALUE    用逗号把一组容器隔开, 以达到对容器的实现. 包括mesos, external, docker在Linux中. 默认: mesos
    --credential=VALUE        一行包含"principal"和"secret"由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是file://path/to/file. 也可以使用路径 file:///path/to/file, 从文件中读取值 。
                             JSON 文件例子:
                             ```
                             {
                               "principal": "username",
                               "secret": "secret"
                             }
                             ```
    --default_container_image=VALUE  当使用外部 containerizer 时，如果没有在一个 task 上指定 ，则使用默认的容器镜像。
    --default_container_info=VALUE  JSON格式的 CONTAINERINFO 将包含到任何没有指定 ContainerInfo 的 ExecutorInfo 中。
                                    例如：
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
    --default_role=VALUE         任何用 --resources 标志位将忽略一个 role ，以及在 --resources 标记位中出现，但被自动检测到的资源。都将使用默认的这个 role。
    --disk_watch_interval=VALUE 周期性时间间隔(例如 10 S ,2 MIN 等)检查硬盘的使用情况。默认是 1 MIN 。
    --docker=VALUE docker       容器化的可执行文件的绝对路径。( 默认: docker )
    --docker_remove_delay=VALUE 移除 docker 前等待的时间 （ 如 3 天，2 周 等）。默认为 6 小时。
    --[no-]docker_kill_orphans  允许 docker kill 掉 orphaned containers 。当你相同的 OS 中启动多个 slave，你应该考虑将此值设为 false 。 以规避 DockerContainerizer 中的一个实例移除被其他 slaves 所启用的 docker 任务。然而，你还应该确保为 slave 启用　checkpoint，这样相同 slave id 可以被重用 。否则当 slave 重启后，docker 任务不会被清除掉。( 默认为　true ) 。
    --docker_sock=VALUE         被挂载到 docker executor 以提供访问 docker daemon 的 docker CLI  的 UNIX socket 路径。它必须是用于使用 slave 的 docker 镜像的路径 （ 默认： /var/run/docker.sock ）。
    --docker_mesos_image=VALUE  docker 镜像用于启动这个 mesos slave 实例。如果一个镜像被指定，docker containerizer 假定 slave 运行在 docker 容器中，并当 slave 重启和恢复时启动 executor 来恢复他们。
    --docker_sandbox_directory=VALUE 描述沙盒在容器中被映射到的绝对路径。（ 默认：  /mnt/mesos/sandbox ）。
    --docker_stop_timeout=VALUE 杀死实例后，在停止它之前 docker  需要等待的间隔时间 （ 默认： 0 Secs ）。
    --[no-]enforce_container_disk_quota 是否为容器启用磁盘限额。这个标记位用来为 ' posix/disk ' 隔离。 （ 默认: false ）。
    --executor_environment_variables     JSON 对象展示必须要传递到 executor 和接下来的 task(s) 中的环境变量。默认情况下 executor 将继承 slave 的环境变量。例如：
                                        ```
                                        {
                                          "PATH": "/bin:/usr/bin",
                                          "LD_LIBRARY_PATH": "/usr/local/lib"
                                        }
                                        ```
    --executor_registration_timeout=VALUE  executor 挂起或者关闭前，等待其注册 slave 的时间。（ 例如，60 S，3 mins 等 ）。默认为 1 MIN 。
    --executor_shutdown_grace_period=VALUE 等待 executor 关闭的时间。( 例如, 60 S, 3 mins 等 )。默认为 5 S 。
    --frameworks_home=VALUE                相对于 executor 的路径前缀的 URI 。
    --gc_delay=VALUE                       清理 executor 目录的延迟时间（ 例如，3 天 或 2 周 等）。注意，根据实际可用磁盘的情况，这个值可能会小些（ 默认：1 周 ）。
    --gc_disk_headroom = VALUE             用于调整 executor 目录的最大磁盘空间。计算方法为 gc_delay * max(0.0, (1.0 - gc_disk_headroom - disk usage)) 每个 --disk_watch_interval 期间，gc_disk_headroom 都必须在 0.0 到 1.0 之间。（ 默认：0.1 ）。
    --hadoop_home=VALUE                    Hadoop 的安装路径。
    --hooks=VALUE                          以逗号分隔的需要在 master 内部被安装的一系列钩子模块
    --hostname=VALUE                       slave 的 主机名 。
    --isolation = VALUE                    隔离机制的使用。 例如  ' posix/cpu,posix/mem ', ' cgroups/cpu,cgroups/mem '  或者 network/port_mapping 及 'external' 或者使用 ***--modules*** 标记位来代替隔离模块。注意，这个标记位只用于 Mesos Containerizer （ 默认：posix/cpu, posix/mem ）。
    --launcher_dir=VALUE                   Mesos 二进制目录路径 （ 默认： /usr/local/lib/mesos ）。
    --modules=VALUE                        待加载的模块列表，并提供给内部的子系统。你也可以使用 file:///path/to/file 或者 /path/to/file 参数值格式从一个文件中读取值。使用 ***--modules="{...}"*** 指定模块内嵌的列表。
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
    --oversubscribed_resources_interval=VALUE  Slave 会定期向 master 更新自己有效的，可以分配的资源。（ 默认：15 S ）。
    --perf_duration=VALUE          一个 perf stat 例子的执行周期。持续时间必须比 perf_interval 少。默认为 10 S。
    --perf_events=VALUE            一系列命令分离的 perf 事件当使用 perf_event 分离器时候来精简每个容器。默认为 None。运行 ' perf list ' 命令查看所有事件。当在 PerfStatistics protobuf 中被通告时候事件名称将被悲观性的消除并使用下划线代替连字符。 例如，cpu-cycles 变为 cpu_cycles。在 PerfStatistics protobuf 中可以看到所有名字。
    --perf_interval = VALUE        多个 perf stat 例子启动间的时间间隔。Perf 示例基于 perf_interval 周期性的被获取并且最近获取的例子将返回而不是按需简化。基于此原因，perf_interval 为独立的资源监控时间间隔。（ 默认： 1 mins ）。
    --qos_controller=VALUE         Qos 控制器的名称被用来超额订阅。
    --qos_correction_interval_min=VALUE  slave 从 Qos 控制器投票和执行 QoS 的更正基于其已运行 tasks 的观察到的性能 这些校正之间的最小间隔有此标记位指定。 （ 默认： 0 secs ）。
    --recover = VALUE                 是否要恢复状态更新以及重新连接 旧的 executor 。" recover " 为有效值。                                             reconnect : 重新连接任何活着的 旧的 executor。
                                      cleanup : 杀死并退出所有活着的 old exetuor 。当你要做一个不兼容的 slave　活着 executor 升级时，可以用这个选项。注意：当 slave 没有设置 checkpointed 时，recovery 不能执行，并且  该 slave 作为一个新的 slave 注册到 master。 ( 默认： reconnect )
    --recovery_timeout=VALUE         分配给 slave 恢复的时间。如果　slave 恢复所用的时间超过 recovery_timeout，将会被终止。( 默认：15 min )
    --registration_backoff_factor=VALUE slave 最初挑选一个随机的时间段[0,B],其中 b = registration_backoff_factor ,( 重新注册 )注册到一个新的 master。 后续重试都基于此时间段成倍扩大 ，最多为 1 mins。( 默认: 1 secs )
    --resource_estimator=VALUE       用于 过度订阅 的 资源评估者 的名称。
    --resource_monitoring_interval=VALUE  executor 资源使用监视周期。（ 例如，10 secs , 1 min 等 ）.( 默认： 1secs )。
    --resources=VALUE                每个 slave 总的可消耗资源。格式：***name(role):value;name(role):value...***。
    --[no-]revocable_cpu_low_priority 通过 revocable CPU 以相对低的优先级运行 containers 。 目前只支持 cgroups/cpu isolator 。( 默认: true )
    --slave_subsystems=VALUE         一系列的逗号分隔的 cgroup 子系统来从二进制运行slave。例如，***memory*** , ***cpuacct*** 。默认为 none 。此功能用于资源的监视以及 no cgroup 下限制设置，它们从 root mesos cgroup 继承而来。
    --[no-]strict                    如果 strict    = true，任何以及所有错误恢复都被认为是致命的。反之，恢复期间，任何预期的错误都会被忽略。 ( 默认： true )
    --[no-]switch_user               是否用提交它们的用户来运行 tasks 而不是使用运行 slave 的用户. ( 需要 setuid 权限)。 ( 默认: true )
    --fetcher_cache_size=VALUE      以字节为单位的 fetcher cache 大小。( 默认: 2 GB )
    --fetcher_cache_dir=VALUE       fetcher cache 的父目录。默认在工作目录中，所以一切都可以在测试环境中保存或删除。然而，典型的生产方案使用的是单独的 缓存卷 。首先，它不意味着需要被备份。其次，要避免沙盒目录和缓存目录以不可预知的方式通过共享空间相互干扰。因此建议，明确设置缓存目录。 ( 默认: /tmp/mesos/fetch )
    --work_dir=VALUE                framework 工作目录的路径。( 默认: /tmp/mesos )

#####当配置了 ' –with-network-isolator ', 以下标记位才会生效：

    --ephemeral_ports_per_container=VALUE   有网络隔离器分配临时端口给一个容器。此端口号必须是 2 的倍数。( 默认: 1024 )
    --eth0_name = VALUE             公网接口的名称 ( 如 eth0 )。如果没有指定，网络隔离器会尝试基于主机的默认网关来猜测它。
    --lo_name=VALUE                 网络 loopback 接口的名称( 例如, lo )。如果没有指定，网络隔离器会尝试猜测它。
    --egress_rate_limit_per_container=VALUE 每个容器的出口流量限制，单位是 字节/每秒。如果没有指定或指定为零，网络隔离器不会强制限制容器的出口流量。这个标记使用字节类型, 定义在 stout 。
    --[no-]network_enable_socket_statistics_summary 是否从每个容器收集 socket 统计摘要。这个标记位被用在  'network/port_mapping' 隔离器。 ( 默认: false )
    --[no-]network_enable_socket_statistics_details 是否从每个容器收集 socket 细节信息。该标记用于 ' network/port_mapping ' 隔离器。( 默认: false )


###Mesos构建配置选项

####配置脚本有以下可选的功能标志

    --enable-shared[=PKGS]    建立共享库 [ 默认: yes ]
    --enable-static[=PKGS]    建立静态库 [ 默认: yes ]
    --enable-fast-install[=PKGS] 快速安装 [ 默认: yes ]
    --disable-libtool-lock    避免锁死 ( 可能会破坏并行编译 )
    --disable-java            不需要构建 Java 绑定
    --disable-python          不需要构建 Python 绑定
    --enable-debug            启用 Debug 调试。如果设置了 CFLAGS/CXXFLAGS ，这个选项不会改变它们。默认: no 
    --enable-optimize         启用优化 如果设置了 CFLAGS/CXXFLAGS ，这个选项不会改变它们。默认: no 
    --disable-bundled         预装相关依赖，而不是绑定 libraries 。
    --disable-bundled-distribute 不包含构建和使用绑定的分发包来替换在 PYTHONPATH 中的已安装版本。
    --disable-bundled-pip	excludes 不包含构建和使用绑定的 pip 包来替换在 PYTHONPATH 中的已安装版本.
    --disable-bundled-wheel	  不包含构建和使用绑定的 wheel 包来替换在 PYTHONPATH 中的已安装版本
    --disable-python-dependency-install 当在 make install 期间安装 python 包，不需要外部的依赖或安装。 

####配置脚本有以下的标志位针对可选的包选项：

    --with-gnu-ld            假设 C 编译器使用 GNU ld [ 默认: no ]
    --with-sysroot=DIR       在 DIR 中搜索依赖库 ( 或者如果编译器的 sysroot 没有指定 )
    --with-zookeeper[=DIR]	  不包含构建和已构建的 ZooKeeper 包来替代在一个给定的目录为前缀名的地址下的已安装版本。
    --with-leveldb[=DIR]	    不包含构建的和使用与 LevelDB 绑定的包来替代在一个给定的目录为前缀名的地址下的已安装版本。
    --with-glog[=DIR]	       不包含构建的和使用与 glog 绑定的包来替代在一个给定的目录为前缀名的地址下的已安装版本。
    --with-protobuf[=DIR]	   不包含构建的和使用与 protobuf 绑定的包来替代在一个给定的目录为前缀名的地址下的已安装版本。
    --with-gmock[=DIR]	      不包含构建的和使用与 gmock 绑定的包来替代在一个给定的目录为前缀名的地址下的已安装版本
    --with-curl=[=DIR] 指定在哪里找到 curl 库
    --with-sasl=[=DIR] 指定在哪里找到 sasl2 库
    --with-zlib=[=DIR] 指定在哪里找到 zlib 库
    --with-apr=[=DIR] 指定在哪里找到 apr-1 库
    --with-svn=[=DIR] 指定在哪里找到 svn-1 库
    --with-network-isolator 建立网络隔离

####一些 influential 的环境变量配置脚本：

使用这些变量来重写 'configure'生成的选择项或帮助其来查找库文件和非标准的名字/地址一起来编程。 

       JAVA_HOME              JDK 根目录
       JAVA_CPPFLAGS          JNI 目录
       JAVA_JVM_LIBRARY       libjvm.so 的完整路径
       MAVEN_HOME             mvn 的根目录
       PROTOBUF_JAR　         在 prefixed builds　上完整的 protobuf jar　路径
       PYTHON　               指定使用哪个 Python 解析器
       PYTHON_VERSION         已经安装并使用的 Python 版本号 例如，'2.3'。该字符串将追加到 Python 解析器
