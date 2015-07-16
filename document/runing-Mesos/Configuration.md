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

    --framework_sorter=VALUE     Policy to use for allocating resources between a given user's frameworks. Options are the same as for user_allocator. (default: drf)

--hooks=VALUE 安装在 master 内的钩子模块（hook module）列表，名子以逗号进行分隔

--hostname=VALUE 这个 master 在 zookeeper 里登记的主机名。如果没有设置，将从绑定的IP地址进行解析。

--[no-]log_auto_initialize Whether to automatically initialize the replicated log used for the registry. If this is set to false, the log has to be manually initialized when used for the very first time. (default: true)

--max_slave_ping_timeouts=VALUE	 master 尝试 ping slave 的最高连续失败次数，超过这个限制的 slave 将被移除。（默认：5）

--modules 需要加载的模块列表，它们可以被内部的子系统调用。使用 —modules=filepath  指定包含模块列表的文件（JSON格式）。可以使用 file:///path/to/file 或者 /path/to/file 来指定文件，或者直接用 —modules=“{...}” 参数指定模块列表。
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

--offer_timeout=VALUE Duration of time before an offer is rescinded from a framework.
This helps fairness when running frameworks that hold on to offers, or frameworks that accidentally drop offers.

--rate_limits=VALUE The value could be a JSON formatted string of rate limits or a file path containing the JSON formatted rate limits used for framework rate limiting.
Remember you can also use the file:///path/to/file or /path/to/file argument value format to write the JSON in a file.

See the RateLimits protobuf in mesos.proto for the expected format.

Example:

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

--recovery_slave_removal_limit=VALUE For failovers, limit on the percentage of slaves that can be removed from the registry *and* shutdown after the re-registration timeout elapses. If the limit is exceeded, the master will fail over rather than remove the slaves.
This can be used to provide safety guarantees for production environments. Production environments may expect that across Master failovers, at most a certain percentage of slaves will fail permanently (e.g. due to rack-level failures).

Setting this limit would ensure that a human needs to get involved should an unexpected widespread failure of slaves occur in the cluster.

Values: [0%-100%] (default: 100%)

--registry=VALUE 注册表持久化策略。可用选项有 'replicated_log', 'in_memory'（用于测试）。默认：replicated_log。

--registry_fetch_timeout=VALUE Duration of time to wait in order to fetch data from the registry after which the operation is considered a failure. (default: 1mins)

--registry_store_timeout=VALUE Duration of time to wait in order to store data in the registry after which the operation is considered a failure. (default: 5secs)

--[no-]registry_strict Whether the Master will take actions based on the persistent information stored in the Registry. Setting this to false means that the Registrar will never reject the admission, readmission, or removal of a slave. Consequently, 'false' can be used to bootstrap the persistent state on a running cluster.
NOTE: This flag is *experimental* and should not be used in production yet. (default: false)

--roles=VALUE A comma separated list of the allocation roles that frameworks in this cluster may belong to.

--[no-]root_submissions 可以root提交框架吗。默认：true

--slave_reregister_timeout=VALUE 超时时间内所有的从节点预计将重新注册时，选举出一个新的master。如果超时没有重新注册上从节点将从注册表里面删除并关闭如果他们试图与主服务器通信。

--user_sorter=VALUE 用户之间资源分配策略，可能是dominant_resource_fairness (drf)。默认：drf

--webui_dir=VALUE 管理页面的网页文件的目录，默认：/usr/local/share/mesos/webui

--weights=VALUE 逗号分割的角色/权重列表，成对表单 'role=weight,role=weight'。 weights是用来表达优先级。

--whitelist=VALUE A filename which contains a list of slaves (one per line) to advertise offers for. The file is watched, and periodically re-read to refresh the slave whitelist. By default there is no whitelist / all machines are accepted. (default: None)
Example:
file:///etc/mesos/slave_whitelist

--zk_session_timeout=VALUE zookeeper 的 session 超时时长。



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
