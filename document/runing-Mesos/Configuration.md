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

--credential=VALUE 一行包含"principal"和"secret"由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是file://path/to/file. 也可以使用路径file:///path/to/file, 从文件中读取值
