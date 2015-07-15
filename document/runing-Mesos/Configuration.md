#mesos 配置向导
标签： 配置

mesos master和slave可以通过命令行参数或环境变量来操作一系列的配置选项。一系列的可用选项可以通过运行mesos-master –help或者mesos-slave –help来查看。每个选项可以通过两种方式设置：

 - 通过使用 –-option_name=value将配置选项传递给应用，即可以通过直接指定值或指定一个内部包含值得文件(--opthon_name=file://文件路径).该路径对应当下工作目录可以为绝对或相对路径。
 - 通过设定系统变量  MESOS_OPTION_NAME(选项名称带有对应的MESOS_前缀名)

配置值通常首先通过环境变量接着通过命令行来发现
**重要的配置**
如果你需要特定复杂的需求，当配置Mesos的时候请参考./configure --help 。另外，本文档列举的近视这些选项的一个子集。你可以通过运行命令带有--help标记来找到一个完整的信息源包含你的mesos版本支持哪个标记，例如 mesos-master --help。

**Master和Slave的配置选项**

下面的这些选项可以同时被master和slave所支持。

     标记                       解释
    --ip=VALUE                 监听的地址
    --firewall_rules=VALUE     该值可以为JSON类型字符的rules或一个文件路径包含JSON类型
                               rules在终端防火墙中被使用。路径可以为以下任何形式 
                               file://文件路径 或 /文件路径。
                               期望的类型信息请参考flags.proto中的防火墙信息。
                               比如：
                               {
                                 "disabled_endpoints" : {
                                    "paths" : [
                                      "/files/browse.json",
                                      "/slave(0)/stats.json",
                                         ]
                                     }
                               }
    --[no-]help                输出帮助信息(默认:否)
    --log_dir=VALUE            输出日志文件的地址(没有默认，如果不指定就什么都不写，不会影响输出到stderr)
    --logbufsecs=VALUE         缓冲日志的时间单位 默认：0秒
    --loggin_level=VALUE       输入日志的级别 默认INFO
    --port=VALUE               监听端口(master默认5050,slave默认5051)
    --[no-]quiet               禁用输出日志到sterr (默认:否)
    --[no-]version             显示版本并且退出(默认：否)



**Master配置选项**

必选标记

        标记                        解释   
        --quorum=VALUE              副本的仲裁数量的大小取决于使用'replicated_log'的基本注册表。                      其非常重要的设定此值为大多数的管理节点.比如，quorum > (管理节点总量)/2,如果在独立模式下运行不需要该设置(non-HA)。
        --work_dir=VALUE            注册表中存储持久性信息的地址。
        --zk=VALUE                  zookeeper URL(用于从主节点中选举出来一个首脑)可能为以下之一：
                                    zk://host1:port1,host2:port2,.../path
                                    zk://username:password@host1:port1,host2:port2,.../path
                                    file:///path/to/file
                                    注意：如果在独立模式下运行就不需要该设置(non-HA)。
可选标记：

        标记                        解释 
        --acls=VALUE               值是一个JSON格式的字符类型ACLs，请记住你也可以使用file:///文件路径或/文件路径参数值格式来将该JSON写入文件，针对期望的格式请参考mesos.proto中的ACLs protobuf
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

--credential=VALUE 一行包含"principal"和"secret"由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是file://path/to/file. 也可以使用路径file:///path/to/file, 从文件中读取值. 
                                  json文件样例：
                                  {
                                      "principal": "username",
                                      "secret": "secret"
                                  }

--default_container_image=VALUE 正在使用外部容器机制时, 如果没有指定任务, 默认的容器镜像被使用. 

--default_container_info=VALUE JSON格式的容器信息将会列出一些没有指定的容器信息的任何执行信息. 
                                  例子:
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

--default_role=VALUE 

--disk_watch_interval=VALUE 硬盘使用情况的周期性检查间隔. 默认: 1分钟

--docker=VALUE docker容器中docker执行文件的绝对路径. 默认: docker

--docker_remove_delay=VALUE 删除容器之前的等待时间. 默认: 6小时

--[no-]docker_kill_orphans 杀掉一个孤立的容器. 当你发布多个slave在同一个OS, 为了避免

--docker_sock=VALUE	The UNIX socket path to be mounted into the docker executor container to provide docker CLI access to the docker daemon. This must be the path used by the slave's docker image. (default: /var/run/docker.sock)

--docker_mesos_image=VALUE	The docker image used to launch this mesos slave instance. If an image is specified, the docker containerizer assumes the slave is running in a docker container, and launches executors with docker containers in order to recover them when the slave restarts and recovers.

--docker_sandbox_directory=VALUE	The absolute path for the directory in the container where the sandbox is mapped to. (default: /mnt/mesos/sandbox)

--docker_stop_timeout=VALUE	The time as a duration for docker to wait after stopping an instance before it kills that instance. (default: 0secs)

--[no-]enforce_container_disk_quota	Whether to enable disk quota enforcement for containers. This flag is used for the 'posix/disk' isolator. (default: false)

--executor_environment_variables	JSON object representing the environment variables that should be passed to the executor, and thus subsequently task(s). By default the executor will inherit the slave's environment variables.
```
 Example:
{
  "PATH": "/bin:/usr/bin",
  "LD_LIBRARY_PATH": "/usr/local/lib"
}
```

--executor_registration_timeout=VALUE	Amount of time to wait for an executor to register with the slave before considering it hung and shutting it down (e.g., 60secs, 3mins, etc) (default: 1mins)

--executor_shutdown_grace_period=VALUE	Amount of time to wait for an executor to shut down (e.g., 60secs, 3mins, etc) (default: 5secs)

--frameworks_home=VALUE	Directory path prepended to relative executor URIs (default: )

--gc_delay=VALUE	Maximum amount of time to wait before cleaning up executor directories (e.g., 3days, 2weeks, etc).
Note that this delay may be shorter depending on the available disk usage. (default: 1weeks)

--gc_disk_headroom=VALUE	Adjust disk headroom used to calculate maximum executor directory age. Age is calculated by:
gc_delay * max(0.0, (1.0 - gc_disk_headroom - disk usage)) every --disk_watch_interval duration. gc_disk_headroom must be a value between 0.0 and 1.0 (default: 0.1)

--hadoop_home=VALUE	Path to find Hadoop installed (for fetching framework executors from HDFS) (no default, look for HADOOP_HOME in environment or find hadoop on PATH) (default: )

--hooks=VALUE	A comma separated list of hook modules to be installed inside master.

--hostname=VALUE	The hostname the slave should report.
If left unset, the hostname is resolved from the IP address that the slave binds to.

--isolation=VALUE	Isolation mechanisms to use, e.g., 'posix/cpu,posix/mem', or 'cgroups/cpu,cgroups/mem', or network/port_mapping (configure with flag: --with-network-isolator to enable), or 'external', or load an alternate isolator module using the --modules flag. Note that this flag is only relevant for the Mesos Containerizer. (default: posix/cpu,posix/mem)

--launcher_dir=VALUE	Directory path of Mesos binaries (default: /usr/local/lib/mesos)

--modules=VALUE	List of modules to be loaded and be available to the internal subsystems.
Remember you can also use the file:///path/to/file or /path/to/file argument value format to have the value read from a file.

Use --modules="{...}" to specify the list of modules inline.

JSON file example:
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

--oversubscribed_resources_interval=VALUE	The slave periodically updates the master with the current estimation about the total amount of oversubscribed resources that are allocated and available. The interval between updates is controlled by this flag. (default: 15secs)

--perf_duration=VALUE	Duration of a perf stat sample. The duration must be less that the perf_interval. (default: 10secs)

--perf_events=VALUE	List of command-separated perf events to sample for each container when using the perf_event isolator. Default is none.
Run command 'perf list' to see all events. Event names are sanitized by downcasing and replacing hyphens with underscores when reported in the PerfStatistics protobuf, e.g., cpu-cycles becomes cpu_cycles; see the PerfStatistics protobuf for all names.

--perf_interval=VALUE	Interval between the start of perf stat samples. Perf samples are obtained periodically according to perf_interval and the most recently obtained sample is returned rather than sampling on demand. For this reason, perf_interval is independent of the resource monitoring interval (default: 1mins)

--qos_controller=VALUE	The name of the QoS Controller to use for oversubscription.

--qos_correction_interval_min=VALUE	The slave polls and carries out QoS corrections from the QoS Controller based on its observed performance of running tasks. The smallest interval between these corrections is controlled by this flag. (default: 0secs)

--recover=VALUE	Whether to recover status updates and reconnect with old executors.
Valid values for 'recover' are

reconnect: Reconnect with any old live executors.

cleanup : Kill any old live executors and exit.

Use this option when doing an incompatible slave or executor upgrade!).

NOTE: If checkpointed slave doesn't exist, no recovery is performed and the slave registers with the master as a new slave. (default: reconnect)

--recovery_timeout=VALUE	Amount of time alloted for the slave to recover. If the slave takes longer than recovery_timeout to recover, any executors that are waiting to reconnect to the slave will self-terminate.
NOTE: This flag is only applicable when checkpoint is enabled. (default: 15mins)

--registration_backoff_factor=VALUE	Slave initially picks a random amount of time between [0, b], where b = registration_backoff_factor, to (re-)register with a new master.
Subsequent retries are exponentially backed off based on this interval (e.g., 1st retry uses a random value between [0, b * 2^1], 2nd retry between [0, b * 2^2], 3rd retry between [0, b * 2^3] etc) up to a maximum of 1mins (default: 1secs)

--resource_estimator=VALUE	The name of the resource estimator to use for oversubscription.

--resource_monitoring_interval=VALUE	Periodic time interval for monitoring executor resource usage (e.g., 10secs, 1min, etc) (default: 1secs)

--resources=VALUE	Total consumable resources per slave, in the form
name(role):value;name(role):value....

--[no-]revocable_cpu_low_priority	Run containers with revocable CPU at a lower priority than normal containers (non-revocable cpu). Currently only supported by the cgroups/cpu isolator. (default: true)

--slave_subsystems=VALUE	List of comma-separated cgroup subsystems to run the slave binary in, e.g., memory,cpuacct. The default is none. Present functionality is intended for resource monitoring and no cgroup limits are set, they are inherited from the root mesos cgroup.

--[no-]strict	If strict=true, any and all recovery errors are considered fatal.
If strict=false, any expected errors (e.g., slave cannot recover information about an executor, because the slave died right before the executor registered.) during recovery are ignored and as much state as possible is recovered. (default: true)

--[no-]switch_user	Whether to run tasks as the user who submitted them rather than the user running the slave (requires setuid permission) (default: true)

--fetcher_cache_size=VALUE	Size of the fetcher cache in Bytes. (default: 2 GB)

--fetcher_cache_dir=VALUE	Parent directory for fetcher cache directories (one subdirectory per slave). By default this directory is held inside the work directory, so everything can be deleted or archived in one swoop, in particular during testing. However, a typical production scenario is to use a separate cache volume. First, it is not meant to be backed up. Second, you want to avoid that sandbox directories and the cache directory can interfere with each other in unpredictable ways by occupying shared space. So it is recommended to set the cache directory explicitly. (default: /tmp/mesos/fetch)

--work_dir=VALUE	Directory path to place framework work directories (default: /tmp/mesos)

Flags available when configured with ‘–with-network-isolator’

--ephemeral_ports_per_container=VALUE	Number of ephemeral ports allocated to a container by the network isolator. This number has to be a power of 2. (default: 1024)

--eth0_name=VALUE	The name of the public network interface (e.g., eth0). If it is not specified, the network isolator will try to guess it based on the host default gateway.

--lo_name=VALUE	The name of the loopback network interface (e.g., lo). If it is not specified, the network isolator will try to guess it.

--egress_rate_limit_per_container=VALUE	The limit of the egress traffic for each container, in Bytes/s. If not specified or specified as zero, the network isolator will impose no limits to containers' egress traffic throughput. This flag uses the Bytes type, defined in stout.

--[no-]network_enable_socket_statistics_summary	Whether to collect socket statistics summary for each container. This flag is used for the 'network/port_mapping' isolator. (default: false)

--[no-]network_enable_socket_statistics_details	Whether to collect socket statistics details (e.g., TCP RTT) for each container. This flag is used for the 'network/port_mapping' isolator. (default: false)


###Mesos Build Configuration Options

####The configure script has the following flags for optional features:

--enable-shared[=PKGS]	build shared libraries [default=yes]

--enable-static[=PKGS]	build static libraries [default=yes]

--enable-fast-install[=PKGS]	optimize for fast installation [default=yes]

--disable-libtool-lock	avoid locking (might break parallel builds)

--disable-java	don't build Java bindings

--disable-python	don't build Python bindings

--enable-debug	enable debugging. If CFLAGS/CXXFLAGS are set, this option won't change them default: no

--enable-optimize	enable optimizations. If CFLAGS/CXXFLAGS are set, this option won't change them default: no

--disable-bundled	build against preinstalled dependencies instead of bundled libraries

--disable-bundled-distribute	excludes building and using the bundled distribute package in lieu of an installed version in PYTHONPATH

--disable-bundled-pip	excludes building and using the bundled pip package in lieu of an installed version in PYTHONPATH

--disable-bundled-wheel	excludes building and using the bundled wheel package in lieu of an installed version in PYTHONPATH

--disable-python-dependency-install	when the python packages are installed during make install, no external dependencies are downloaded or installed

####The configure script has the following flags for optional packages:

--with-gnu-ld	assume the C compiler uses GNU ld [default=no]

--with-sysroot=DIR	Search for dependent libraries within DIR (or the compiler's sysroot if not specified).

--with-zookeeper[=DIR]	excludes building and using the bundled ZooKeeper package in lieu of an installed version at a location prefixed by the given path

--with-leveldb[=DIR]	excludes building and using the bundled LevelDB package in lieu of an installed version at a location prefixed by the given path

--with-glog[=DIR]	excludes building and using the bundled glog package in lieu of an installed version at a location prefixed by the given path

--with-protobuf[=DIR]	excludes building and using the bundled protobuf package in lieu of an installed version at a location prefixed by the given path

--with-gmock[=DIR]	excludes building and using the bundled gmock package in lieu of an installed version at a location prefixed by the given path

--with-curl=[=DIR]	specify where to locate the curl library

--with-sasl=[=DIR]	specify where to locate the sasl2 library

--with-zlib=[=DIR]	specify where to locate the zlib library

--with-apr=[=DIR]	specify where to locate the apr-1 library

--with-svn=[=DIR]	specify where to locate the svn-1 library

--with-network-isolator	builds the network isolator

####Some influential environment variables for configure script:

通过这些 name / locations 变量可以很快定位到对应的库或程序。

JAVA_HOME	Java Development Kit 本地路径 (JDK)

JAVA_CPPFLAGS	JNI 预编译器标记

JAVA_JVM_LIBRARY	libjvm.so 的完整路径

MAVEN_HOME		用来定位 mvn 在 MAVEN_HOME/bin/mvn 目录中

PROTOBUF_JAR	protobuf jar 完整路径 on prefixed builds

PYTHON	Python 解析器路径

PYTHON_VERSION	已安装并使用的 Python 版本, for example '2.3'. This string will be appended to the Python interpreter canonical name.




 







