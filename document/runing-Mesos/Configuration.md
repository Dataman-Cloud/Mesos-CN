## mesos 配置向导

mesos master 和 slave 可以通过命令行参数或环境变量来传递一系列的配置选项。通过运行 `mesos-master --help` 或者 `mesos-slave --help` 可以查看相关的可用选项。每个选项可以通过以下两种方式设置：

 - 执行命令的时候使用 –-option_name=value 来传递配置选项。value 既可以是数值，也可以指定包含参数的文件 (--opthon_name=file://文件路径)。 该路径既可以是绝对路径，也可以是相对当前工作目录的相对路径。

 - 通过设定环境变量 MESOS_OPTION_NAME (变量名都以 MESOS_ 开头)  
执行时会先读取环境变量，然后才看命令行参数

配置参数会首先在环境中搜索，然后才是命令行。

####重要的配置
如果你有特定的需求，当配置 Mesos 的时候请参考 ./configure --help。另外，本文档列举了最新的配置选项。如果你想知道手头的版本支持哪些标志位，你可以运行带有 --help 的命令，例如 mesos-master --help。

### Master 和 Slave 的配置选项

下列选项 都被 master 和 slave 所支持：

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --advertise_ip=VALUE
  </td>
  <td>
用来触达 mesos master/slave 的 IP 广播地址。
Mesos master/slave 不会与这个 IP 地址绑定。
但是，这个 IP 地址可以被用来访问 Mesos master/slave.
  </td>
</tr>
<tr>
  <td>
    --advertise_port=VALUE
  </td>
  <td>
用来触达 mesos master/slave 的广播端口 (配合
<code>advertise_ip</code>). Mesos master/slave 不与这个端口绑定。
但是，这个端口 (配合 <code>advertise_ip</code>) 可以用于访问 Mesos master/slave.
  </td>
</tr>
<tr>
  <td>
    --firewall_rules=VALUE
  </td>
  <td>
  该值是终端防火墙的规则（rules），可以为JSON 类型的 rules 或包含 JSON 类型 rules 的文件。文件路径可以为       <code>file:///path/to/file</code> 或者 <code>/path/to/file</code>。<p/>
  规则的格式请参考文件 <code>flags.proto</code> 中的 <code>Firewall</code> 信息。
<p/>
例如:
<pre><code>{
  "disabled_endpoints" : {
    "paths" : [
      "/files/browse",
      "/metrics/snapshot"
    ]
  }
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --[no-]help
  </td>
  <td>
输出帮助信息 (默认值: false)
  </td>
</tr>
<tr>
  <td>
    --ip=VALUE
  </td>
  <td>
监听的 IP 地址. 这个不能与<code>--ip_discovery_command</code>一起使用. (master默认5050，slave默认5051)
  </td>
</tr>
<tr>
  <td>
    --ip_discovery_command=VALUE
  </td>
  <td>
IP 发现可选项: 如果设置 IP 地址，master/slave 将会尝试绑定这个 IP 地址。
不能与 <code>--ip</code> 一起使用.
  </td>
</tr>
<tr>
  <td>
    --port=VALUE
  </td>
  <td>
监听端口
  </td>
</tr>
<tr>
  <td>
    --[no-]version
  </td>
  <td>
显示版本并退出 (默认: false)
  </td>
</tr>
<tr>
  <td>
    --hooks=VALUE
  </td>
  <td>
  一个由逗号分隔的 hook 模块列表将被安装到 master/slave。
  </td>
</tr>
<tr>
  <td>
    --hostname=VALUE
  </td>
  <td>
  slave 节点报告或 master 节点在 ZooKeeper 里广播的 hostname.
  如果不做设置，hostname 将解析为 master/slave 绑定的 IP 地址。
  除非用户已经使用 <code>--no-hostname_lookup</code> 明确禁止了此功能, in which case the IP itself
is used.
  </td>
</tr>
<tr>
  <td>
    --[no-]hostname_lookup
  </td>
  <td>
  当没有明确指定 hostname 时(例如 <code>--hostname</code>)，是否查询找出服务器的 hostname。
  默认值是 true; 如果设置为 <code>false</code> Mesos 将会使用 IP 地址信息，除非 hostname 被明确指出了。
  (默认值: true)
  </td>
</tr>
<tr>
  <td>
    --modules=VALUE
  </td>
  <td>
List of modules to be loaded and be available to the internal
subsystems.
<p/>
Use <code>--modules=filepath</code> to specify the list of modules via a
file containing a JSON-formatted string. <code>filepath</code> can be
of the form <code>file:///path/to/file</code> or <code>/path/to/file</code>.
<p/>
Use <code>--modules="{...}"</code> to specify the list of modules inline.
<p/>
Example:
<pre><code>{
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
}</code></pre>
  </td>
</tr>
</table>

*masters 和 slaves 同时支持以下这些日志选项* 更多日志信息，请访问 [http://mesos.apache.org/documentation/latest/logging/](http://mesos.apache.org/documentation/latest/logging/)

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --[no-]quiet
  </td>
  <td>
禁用输出日志到 sterr （默认:false）
  </td>
</tr>
<tr>
  <td>
    --log_dir=VALUE
  </td>
  <td>
  输出日志文件的位置。默认方式下，不生成日志文件。这个参数不影响输出到 stderr 的日志。
  如果特别指定了，就可以通过 Mesos webUI 看到这个日志文件。
<b>注意</b>: 第三方日志信息 (比如，ZooKeeper) 将只能写入到 stderr!
  </td>
</tr>
<tr>
  <td>
    --logbufsecs=VALUE
  </td>
  <td>
缓冲日志的时长（秒数）默认：0秒
  </td>
</tr>
<tr>
  <td>
    --logging_level=VALUE
  </td>
  <td>
  输出日志的起始级别，包括 <code>INFO</code>, <code>WARNING</code>, <code>ERROR</code>。如果使用了<code>--quiet</code> 标记，只会影响到输出到 <code>--log_dir</code> 的日志的级别（默认：INFO）
  </td>
</tr>
<tr>
  <td>
    --[no-]initialize_driver_logging
  </td>
  <td>
  master/slave 是否为 Mesos scheduler 和 executor driver 初始化 Google logging.
  scheduler/executor drivers 将分别记录日志，不会写入 master/slave 的日志中。
<p/>
如果使用的是 HTTP scheduler/executor APIs，这个选项将无效。
（默认：true）
  </td>
</tr>
<tr>
  <td>
    --external_log_file=VALUE
  </td>
  <td>
  定位外部管理的日志文件位置。Mesos 不会直接写入这个文件，仅会通过 WebUI 和 HTTP API 将其
  暴露出来。这个仅用于混合外部日志机制来记录日志到 stderr 的情况。比如，syslog 或 journald。
<p/>
当通过 <code>--quiet</code> 指定后，这个选项将无效。
<p/>
此选项在 WebUI 的优先级高于 <code>--log_dir</code> . 但即使这个选项被指定了，日志任然会被
写入到 <code>--log_dir</code>。
  </td>
</tr>
</table>


##Master 配置选项

*必选参数*

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --quorum=VALUE
  </td>
  <td>
  使用基于 replicated-Log 的注册表时，复制的个数。
  此值需要设置为masters总数量的一半以上，也就是：<code>quorum > (number of masters)/2</code>。
  注意：单机模式下不需要设置此参数。（非HA模式）
  </td>
</tr>
<tr>
  <td>
    --work_dir=VALUE
  </td>
  <td>
  Registry 中持久化信息存储的位置。（如：<code>/var/lib/mesos/master</code>）
  </td>
</tr>
<tr>
  <td>
    --zk=VALUE
  </td>
  <td>
  ZooKeeper 的 URL地址 （用于在masters中做领导选举）可能是下面所列形式中的一种：
<pre><code>zk://host1:port1,host2:port2,.../path
zk://username:password@host1:port1,host2:port2,.../path
file:///path/to/file (where file contains one of the above)</code></pre>
<b>注意</b>: 单机模式下不需要设置此参数。（非HA模式）.
  </td>
</tr>
</table>

*可选参数*

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --acls=VALUE
  </td>
  <td>
  此参数用于认证。一般是 JSON 格式的 ACLs 的字符串或者文件。
  路径一般是这样的格式：<code>file:///path/to/file</code> 或 <code>/path/to/file</code>
<p/>
  注意：如果参数 <code>--authorizers</code> 的值与 <code>local</code> 的不相同，ACLs 的内容将被忽略。
<p/>
  在 authorizer.proto 中查看 ACLs protobuf 参考格式。
<p/>
举例:
<pre><code>{
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
  "teardown_frameworks": [
    {
      "principals": { "values": ["a", "b"] },
      "framework_principals": { "values": ["c"] }
    }
  ],
  "set_quotas": [
    {
      "principals": { "values": ["a"] },
      "roles": { "values": ["a", "b"] }
    }
  ],
  "remove_quotas": [
    {
      "principals": { "values": ["a"] },
      "quota_principals": { "values": ["a"] }
    }
  ]
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --allocation_interval=VALUE
  </td>
  <td>
  （批次）执行分配（allocations）的间隔时间。（如：500ms,1秒……）
  默认值：1秒
  </td>
</tr>
<tr>
  <td>
    --allocator=VALUE
  </td>
  <td>
  分配器，用于给框架分配资源。默认使用 <code>HierarchicalDRF</code> 分配器，也可以通过
  <code>--modules</code> 模块来选择其他的分配器。
  （默认值：HierarchicalDRF）
  </td>
</tr>
<tr>
  <td>
    --[no-]authenticate
  </td>
  <td>
  如果是 <code>true</code>，则只有认证过的框架可以注册。
  如果是 <code>false</code>，则未认证的框架也可以注册。（默认：false）
  </td>
</tr>
<tr>
  <td>
    --[no-]authenticate_http
  </td>
  <td>
  如果是 <code>true</code>，则只有支持认证机制的已认证的 HTTP endpoints 请求被允许访问。
  如果是 <code>false</code>，则未认证的 HTTP endpoint 请求也会被允许访问。
  （默认：false）
  </td>
</tr>
<tr>
  <td>
    --[no-]authenticate_slaves
  </td>
  <td>
  如果是 <code>true</code>，只有认证过的 slaves 才能注册。
  如果是 <code>false</code>，未认证的 slaves 也可以注册。
  （默认：false）
  </td>
</tr>
<tr>
  <td>
    --authenticators=VALUE
  </td>
  <td>
  框架或 slave 进行认证时使用的认证器。默认是 <code>crammd5</code>，也可以通过使用
  <code>--modules</code> 更换其他认证模块。
  （默认：crammd5）
  </td>
</tr>
<tr>
  <td>
    --authorizers=VALUE
  </td>
  <td>
  用于进行授权的 Authorizer。默认使用 <code>local</code>,也可以通过使用
  <code>--modules</code> 替换成其他的 authorizer。
<p/>
  注意：如果参数 <code>--authorizers</code> 提供了一个与 <code>local</code> 不一样的值。
  则通过<code>--acls</code> 设置的 ACLs 参数将被忽略。
<p/>
  目前并不支持多个 authorizers. （默认：local）
  </td>
</tr>
<tr>
  <td>
    --cluster=VALUE
  </td>
  <td>
  集群别名，会在 WebUI上显示。
  </td>
</tr>
<tr>
  <td>
    --credentials=VALUE
  </td>
  <td>
  一个存取凭证的路径。这个路径可以指向一个内容为凭证列表的文本文件，在这个文件中每一行包括由空格隔开的<code>principal</code>和<code>secret</code>。也可以指向一个包含凭证信息的 JSON 格式文件。
  路径的格式可以是：<code>file:///path/to/file</code> 或 <code>/path/to/file</code>
JSON 文件举例:
<pre><code>{
  "credentials": [
    {
      "principal": "sherman",
      "secret": "kitesurf"
    }
  ]
}</code></pre>
文本文件举例:
<pre><code>username secret</code></pre>
  </td>
</tr>
<tr>
  <td>
    --framework_sorter=VALUE
  </td>
  <td>
  给定 framework 之间的资源分配策略。选项与 user_allocator 相同。
  （默认：drf）
  </td>
</tr>
<tr>
  <td>
    --http_authenticators=VALUE
  </td>
  <td>
  HTTP 认证器用于处理已验证的 endpoints 的请求。默认值是 <code>basic</code>，或者通过
  <code>--modules</code> 加载一个其他的 HTTP 认证器。
<p/>
目前不支持多种 HTTP 认证器。（默认：basic）
  </td>
</tr>
<tr>
  <td>
    --[no-]log_auto_initialize
  </td>
  <td>
  是否自动初始化注册使用的 *replicated log* 。如果设置为否，日志将在每次使用时手动初始化。
  （默认值：true）
  </td>
</tr>
<tr>
  <td>
    --max_completed_frameworks=VALUE
  </td>
  <td>
  存储在内存中的完成框架的最大数量。（默认：50）
  </td>
</tr>
<tr>
  <td>
    --max_completed_tasks_per_framework
=VALUE
  </td>
  <td>
  存储在内存的每个框架中已完成任务的最大数量。（默认：1000）
  </td>
</tr>
<tr>
  <td>
    --max_slave_ping_timeouts=VALUE
  </td>
  <td>
  一个 slave 对于master的 ping 响应失败的最大次数。
  如果 slaves 没有在 <code>max_slave_ping_timeouts</code> 之内响应，就会尝试关机。
  （默认：5）
  </td>
</tr>
<tr>
  <td>
    --offer_timeout=VALUE
  </td>
  <td>
  一个 offer 撤销的超时时间。
  这可以让不同的 frameworks 提供的 offer 获得更公平的响应。
  如果不设置， offers 没有超时限制。
  </td>
</tr>
<tr>
  <td>
    --rate_limits=VALUE
  </td>
  <td>
  该值可以为一个 JSON 格式的速率限制或一个文件路径包含了被 framework 限速 所使用的 JSON 格式的速率限制。请记住你也可以是使用 file:///path/to/file 或 <code>/path/to/file</code> 参数值格式来将该 JSON 写入至一个文件。
  <p/>
  期望的格式请参考 mesos.proto 中的 RateLimits protobuf.
  <p/>
Example:
<pre><code>{
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
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --recovery_slave_removal_limit=VALUE
  </td>
  <td>
  针对故障转移，限制上的百分比的 slaves 可以从注册中移除并关机在重新注册的超时时间到了之后。
  如果该限制被突破， master 将实行故障转移而不是移除 slaves.
  这可被用来针对生产环境提供安全保障。生产环境可能期望在 Master 故障转移过程中， 最多一定百分比的 slaves 将永久性的挂掉 (比如, 由于 rack-level 的故障)。
  设定该限制可以保证一个人需要参与进来当在该集群中一个非预期的大范围的 slave 故障发生。值: [0%-100%] (默认: 100%)
  </td>
</tr>
<tr>
  <td>
    --registry=VALUE
  </td>
  <td>
  注册表持久化策略。可用选项有 <code>replicated_log</code>,<code>in_memory</code>（用于测试）。
  默认：replicated_log。
  </td>
</tr>
<tr>
  <td>
    --registry_fetch_timeout=VALUE
  </td>
  <td>
  在操作被认为是一个失败后的为了从注册中提取数据的等待的时间间隔.(默认： 1mins)
  </td>
</tr>
<tr>
  <td>
    --registry_store_timeout=VALUE
  </td>
  <td>
  等待的时间周期为了当操作被认为一个失败的时候将数据存储入注册机。 (默认：20secs)
  </td>
</tr>
<tr>
  <td>
    --[no-]registry_strict
  </td>
  <td>
  无论 Master 是否将基于注册机中存储的持久信息来采取行动。设定改值为 false 意味着注册员将永远拒绝入列，出列和一个 slave 的移除。所以， <code>false</code> 可以用来在一个运行的集群上来引导持久化的状态。注意： 该标志位是 *experimental* 而且还不能在应用中使用.(默认: false)
  </td>
</tr>
<tr>
  <td>
    --roles=VALUE
  </td>
  <td>
  其 frameworks 在这个集群中可能归属于的用逗号分离的一系列指派的角色。
  </td>
</tr>
<tr>
  <td>
    --[no-]root_submissions
  </td>
  <td>
  root 是否可以提交 frameworks? (默认: true)
  </td>
</tr>
<tr>
  <td>
    --slave_ping_timeout=VALUE
  </td>
  <td>
  在每个 slave 被期望从一个 master 回应一个 ping 值的超时时间。 Slaves 如果不是在 max_slave_ping_timeouts 回复，ping 从新尝试将被移除.  (默认: 15secs)
  </td>
</tr>
<tr>
  <td>
    --slave_removal_rate_limit=VALUE
  </td>
  <td>
  最大的比例(e.g., <code>1/10mins</code>, <code>2/3hrs</code>, etc) 对于那个 slaves 将被从 master 中移除当他们遇到健康检测失败。默认的是 slave 将尽可能快的被移除当它们遇到健康监测失败。值为 <code>(Number of slaves)/(Duration)</code> 的模式。
  </td>
</tr>
<tr>
  <td>
    --slave_reregister_timeout=VALUE
  </td>
  <td>
  在所有的 slaves 被期望重新注册当一个新的 master 被选举为 leader 的超时时间。 Slaves 其不会在此超时时间内被重新注册将被从注册中移除并将被关掉如果它们尝试去与 master 通信。
  注意： 该值将被设置为最少 10mins. (默认: 10mins)
  </td>
</tr>
<tr>
  <td>
    --user_sorter=VALUE
  </td>
  <td>
  被用来在用户中分配资源的策略。可以为以下之一：dominant_resource_fairness (drf) (default: drf)
  </td>
</tr>
<tr>
  <td>
    --webui_dir=VALUE
  </td>
  <td>
  管理页面的网页文件的目录，默认：/usr/local/share/mesos/webui
  </td>
</tr>
<tr>
  <td>
    --weights=VALUE
  </td>
  <td>
  逗号分割的角色/权重列表，成对表单 <code>role=weight,role=weight</code>。 weights是用来表达优先级。
  </td>
</tr>
<tr>
  <td>
    --whitelist=VALUE
  </td>
  <td>
  一个 文件名器包含一系列的 slaves （每行一个）来通告 offers.该文件被观测，并周期性的重读取来刷新 slave 白名单。 默认的这里没有白名单/所有机器被接收. (默认: None)
  文件路径可以是这样的形式：
<code>file:///path/to/file</code> 或 <code>/path/to/file</code>.
  </td>
</tr>
<tr>
  <td>
    --zk_session_timeout=VALUE
  </td>
  <td>
zookeeper 的 session 超时时长。 (默认: 10secs)
  </td>
</tr>
</table>

*通过 `--with-network-isolator` 配置时可用的标记*

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --max_executors_per_slave=VALUE
  </td>
  <td>
  每个 Slave 上最大允许的执行器数量。网络监控和隔离机制强行限制每个执行器使用的端口资源，所以每个 slave 上只能跑一定数量的执行器。
  </td>
</tr>
</table>

## Slave 选项

*必选项*

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --master=VALUE
  </td>
  <td>
  可能是其中的一种：
  <code>host:port</code>
  <code>zk://host1:port1,host2:port2,.../path</code>
  <code>zk://username:password@host1:port1,host2:port2,.../path</code>
  <code>file:///path/to/file</code> (包含以上中的一个)
  </td>
</tr>
</table>

*可选项*

<table class="table table-striped">
  <thead>
    <tr>
      <th width="30%">
        Flag
      </th>
      <th>
        说明
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --appc_store_dir=VALUE
  </td>
  <td>
  appc 提供者存储镜像的目录
(默认: /tmp/mesos/store/appc)
  </td>
</tr>
<tr>
  <td>
    --attributes=VALUE
  </td>
  <td>
  slave 机器的属性,格式为：
  <code>rack:2</code> 或者<code>rack:2;u:1</code>
  </td>
</tr>
<tr>
  <td>
    --authenticatee=VALUE
  </td>
  <td>
  用于主节点身份验证，默认crammd5，或者用<code>-—modules加载备用模块</code>。（默认：crammd5）
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_cpu_enable_pids_and_tids_count
  </td>
  <td>
  Cgroups 的功能标记，可以统计容器内的进程和线程的数量。（默认：false）
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_enable_cfs
  </td>
  <td>
  Cgroups 的功能标记，通过限制CFS带宽来限制CPU资源. (默认: defult)
  </td>
</tr>
<tr>
  <td>
    --cgroups_hierarchy=VALUE
  </td>
  <td>
  cgroups的根路径位置. 默认: /sys/fs/cgroup
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_limit_swap
  </td>
  <td>
  Cgroups 的功能标记，可以对内存和swap进行限制，而不仅限制内存。（默认: false）
  </td>
</tr>
<tr>
  <td>
    --cgroups_net_cls_primary_handle
  </td>
  <td>
  一个非零，16位的句柄。形式类似于：`0xAAAA`. 这将作为 net_cls cgroup 的主句柄来使用。
  </td>
</tr>
<tr>
  <td>
    --cgroups_net_cls_secondary_handles
  </td>
  <td>
  一系列的类似 0xAAAA,0xBBBB 形式的次要句柄，将与主句柄配合使用。只有在设置了
  <code>--cgroups_net_cls_primary_handle</code> 之后，才会生效。
  </td>
</tr>
<tr>
  <td>
    --cgroups_root=VALUE
  </td>
  <td>
  根cgroup的命名. 默认: mesos
  </td>
</tr>
<tr>
  <td>
    --container_disk_watch_interval=VALUE
  </td>
  <td>
  用于查询容器中磁盘配额的时间间隔. 被用于<code>posix/disk</code>的时间间隔, 默认: 15秒
  </td>
</tr>
<tr>
  <td>
    --container_logger=VALUE
  </td>
  <td>
  容器日志记录器的名称，日志记录器用来记录容器（如：执行器，任务）的标准输出和错误日志。
  默认的日志记录器将会写入到沙盒目录中的 <code>stdout</code> 和 <code>stderr</code>。
  </td>
</tr>
<tr>
  <td>
    --containerizer_path=VALUE
  </td>
  <td>
  当外部隔离机制被激活时(<code>--isolation=external</code>), 外部容器被执行的路径
  </td>
</tr>
<tr>
  <td>
    --containerizers=VALUE
  </td>
  <td>
  由逗号分隔的容器化实现方式列表。可选项有 <code>mesos</code>, <code>external</code>, and
  <code>docker</code> (on Linux). 排列的顺序就是容器化过程中尝试的顺序。
  （默认：mesos）
  </td>
</tr>
<tr>
  <td>
    --credential=VALUE
  </td>
  <td>
  一行包含<code>principal</code>和<code>secret</code>由空格隔开的文本路径. 或是包含一条凭证的JSON格式文件的路径. 路径的格式是<code>file://path/to/file</code> 或 <code>/path/to/file</code>.
  例如:
<pre><code>{
  "principal": "username",
  "secret": "secret"
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --default_container_image=VALUE
  </td>
  <td>
  当使用外部容器化器时，在任务没有特别指定的情况下所使用的默认容器镜像。
  没有在一个 task 上指定 ，则使用默认的容器镜像。
  </td>
</tr>
<tr>
  <td>
    --default_container_info=VALUE
  </td>
  <td>
  JSON格式的 CONTAINERINFO 将包含到任何没有指定 ContainerInfo 的 ExecutorInfo 中。
<p/>
See the ContainerInfo protobuf in mesos.proto for
the expected format.
<p/>
例如:
<pre><code>{
  "type": "MESOS",
  "volumes": [
    {
      "host_path": "./.private/tmp",
      "container_path": "/tmp",
      "mode": "RW"
    }
  ]
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --default_role=VALUE
  </td>
  <td>
  任何用 <code>--resources </code>标志位将忽略一个 role ，以及在 --resources 标记位中出现，但被自动检测到的资源。都将使用默认的这个 role。
  </td>
</tr>
<tr>
  <td>
    --disk_watch_interval=VALUE
  </td>
  <td>
  周期性时间间隔(例如 10 S ,2 MIN 等)检查slave管理的硬盘使用情况。
  这个会对存档信息和沙盒做垃圾回收。(默认: 1mins)
  </td>
</tr>
<tr>
  <td>
    --docker=VALUE
  </td>
  <td>
  docker容器化的可执行文件的绝对路径。( 默认: docker )
  </td>
</tr>
<tr>
  <td>
    --[no-]docker_kill_orphans
  </td>
  <td>
  允许 docker kill 掉 orphaned containers 。当你相同的 OS 中启动多个 slave，你应该考虑将此值设为 false 。 以规避 DockerContainerizer 中的一个实例移除被其他 slaves 所启用的 docker 任务。然而，你还应该确保为 slave 启用　checkpoint，这样相同 slave id 可以被重用 。否则当 slave 重启后，docker 任务不会被清除掉。( 默认为　true ) 。
  </td>
</tr>
<tr>
  <td>
    --docker_mesos_image=VALUE
  </td>
  <td>
  docker 镜像用于启动这个 mesos slave 实例。如果一个镜像被指定，docker containerizer 假定 slave 运行在 docker 容器中，并当 slave 重启和恢复时启动 executor 来恢复他们。
  </td>
</tr>
<tr>
  <td>
    --docker_registry=VALUE
  </td>
  <td>
  一个下拉 Docker 镜像的默认 url. 可以是一个 Docker registry 服务的 URL（例如：<code>https://registry.docker.io</code>），也可以是 一个包含Docker存档的本地路径（例如：<code>/tmp/docker/images</code>）
  （默认：https://registry-1.docker.io ）
  </td>
</tr>
<tr>
  <td>
    --docker_remove_delay=VALUE
  </td>
  <td>
  移除 docker 前等待的时间 （ 如 3 天，2 周 等）。默认为 6 小时。
  </td>
</tr>
<tr>
  <td>
    --docker_socket=VALUE
  </td>
  <td>
  一个安装在 Docker executor 容器内部的 UNIX 套接字路径。用来提供通过 CLI 访问 docker daemon 的能力。
  这个必须是 slave docker 镜像用的路径。（默认：/var/run/docker.sock）
  </td>
</tr>
<tr>
  <td>
    --docker_stop_timeout=VALUE
  </td>
  <td>
  杀死实例后，在停止它之前 docker  需要等待的间隔时间 （ 默认： 0 Secs ）。
  </td>
</tr>
<tr>
  <td>
    --docker_store_dir=VALUE
  </td>
  <td>
  Docker provisioner 用来存储镜像的目录。（默认：/tmp/mesos/store/docker）
  </td>
</tr>
<tr>
  <td>
    --[no-]enforce_container_disk_quota
  </td>
  <td>
  否为容器启用磁盘限额。这个标记位用来为 <code>posix/disk</code>  隔离。 （ 默认: false ）。
  </td>
</tr>
<tr>
  <td>
    --executor_environment_variables=VALUE
  </td>
  <td>
  使用 JSON 对象格式的环境变量。会通过 executor 来传递之后的 task。
  默认情况下，executor 会继承 slave 的环境变量。
  例如:
<pre><code>{
  "PATH": "/bin:/usr/bin",
  "LD_LIBRARY_PATH": "/usr/local/lib"
}</code></pre>
  </td>
</tr>
<tr>
  <td>
    --executor_registration_timeout=VALUE
  </td>
  <td>
  executor 挂起或者关闭前，等待其注册 slave 的时间。（ 例如，60 S，3 mins 等 ）。默认为 1 MIN 。
  </td>
</tr>
<tr>
  <td>
    --executor_shutdown_grace_period=VALUE
  </td>
  <td>
  等待 executor 关闭的时间。( 例如, 60 S, 3 mins 等 )。默认为 5 S 。
  </td>
</tr>
<tr>
  <td>
    --fetcher_cache_dir=VALUE
  </td>
  <td>
  fetcher cache 的父目录。（每一个slave有一个子目录）。
  （默认：/tmp/mesos/fetch）
  </td>
</tr>
<tr>
  <td>
    --fetcher_cache_size=VALUE
  </td>
  <td>
  以字节为单位的 fetcher cache 大小。( 默认: 2 GB )
  </td>
</tr>
<tr>
  <td>
    --frameworks_home=VALUE
  </td>
  <td>
  相对于 executor 的路径前缀的 URI 。
  </td>
</tr>
<tr>
  <td>
    --gc_delay=VALUE
  </td>
  <td>
  清理 executor 目录的延迟时间（ 例如，3 天 或 2 周 等）。
  注意，根据实际可用磁盘的情况，这个值可能会小些（ 默认：1 周 ）。
  </td>
</tr>
<tr>
  <td>
    <a name="gc_disk_headroom"></a>
    --gc_disk_headroom=VALUE
  </td>
  <td>
Adjust disk headroom used to calculate maximum executor
directory age. Age is calculated by:
<code>gc_delay * max(0.0, (1.0 - gc_disk_headroom - disk usage))</code>
every <code>--disk_watch_interval</code> duration. <code>gc_disk_headroom</code> must
be a value between 0.0 and 1.0 (default: 0.1)
  </td>
</tr>
<tr>
  <td>
    --hadoop_home=VALUE
  </td>
  <td>
  Hadoop 的安装路径。（用于从 HDFS 提取框架 executors ）
  （没有默认项，在环境中查找<code>HADOOP_HOME</code>，或者在<code>PATH</code> 查询 hadoop）
  </td>
</tr>
<tr>
  <td>
    --image_providers=VALUE
  </td>
  <td>
  由逗号分割的支持的镜像供应商列表。如：<code>APPC,DOCKER</code>.
  </td>
</tr>
<tr>
  <td>
    --image_provisioner_backend=VALUE
  </td>
  <td>
  从镜像中提取容器 rootfs 的策略。
  如：<code>bind</code>, <code>copy</code>. (默认: copy)
  </td>
</tr>
<tr>
  <td>
    --isolation=VALUE
  </td>
  <td>
  所采用的隔离机制，如：<code>posix/cpu,posix/mem</code> 或 <code>cgroups/cpu,cgroups/mem</code>
  或 network/port_mapping（通过 <code>--with-network-isolator</code> 标记来开启）
  或 <code>external</code> 或者通过code>--modules</code>标记替换成另一个隔离模块。
  注意：这个标记仅用于 Mesos 容器化器。（默认：posix/cpu,posix/mem）
  </td>
</tr>
<tr>
  <td>
    --launcher=VALUE
  </td>
  <td>
  Mesos 容器化器所使用的启动器。可以是 <code>linux</code> 或 <code>posix</code>。
  Linux 启动器需要cgroups隔离机制。每个隔离器需要 Linux 的 namespaces 如 网络，pid,等。
  如果没有特别指定，slave 将选择一个作为root运行的 Linux 启动器。
  </td>
</tr>
<tr>
  <td>
    --launcher_dir=VALUE
  </td>
  <td>
  Mesos 二进制目录路径。 Mesos 可以在这个目录下找到 健康检查，fetcher，容器化器，executor
  的二进制文件。（ 默认： /usr/local/lib/mesos ）。
  </td>
</tr>
<tr>
  <td>
    --oversubscribed_resources_interval=VALUE
  </td>
  <td>
  Slave 会定期向 master 更新自己有效的，可以分配的资源。
  更新的间隔时间是由这个 flag 控制的。
  （ 默认：15 S ）。
  </td>
</tr>
<tr>
  <td>
    --perf_duration=VALUE
  </td>
  <td>
  一个 perf stat 例子的执行周期。持续时间必须比 perf_interval 少。（默认： 10 secs）
  </td>
</tr>
<tr>
  <td>
    --perf_events=VALUE
  </td>
  <td>
  一系列命令分离的 perf 事件当使用 perf_event 分离器时候来精简每个容器。默认为 None。运行 <code> perf list </code>命令查看所有事件。当在 PerfStatistics protobuf 中被通告时候事件名称将被悲观性的消除并使用下划线代替连字符。 例如，<code>cpu-cycles</code> 变为 <code>cpu_cycles</code>。在 PerfStatistics protobuf 中可以看到所有名字。
  </td>
</tr>
<tr>
  <td>
    --perf_interval=VALUE
  </td>
  <td>
Interval between the start of perf stat samples. Perf samples are
obtained periodically according to <code>perf_interval</code> and the most
recently obtained sample is returned rather than sampling on
demand. For this reason, <code>perf_interval</code> is independent of the
resource monitoring interval. (default: 60secs)
  </td>
</tr>
<tr>
  <td>
    --qos_controller=VALUE
  </td>
  <td>
  Qos 控制器的名称被用来超额订阅。
  </td>
</tr>
<tr>
  <td>
    --qos_correction_interval_min=VALUE
  </td>
  <td>
  slave 从 Qos 控制器投票和执行 QoS 的更正基于其已运行 tasks 的观察到的性能 这些校正之间的最小间隔有此标记位指定。 （ 默认： 0 secs ）。
  </td>
</tr>
<tr>
  <td>
    --recover=VALUE
  </td>
  <td>
  是否恢复更新状态并与老的 executors 重新连接。
  <code>recover</code> 可用的值有：
  reconnect：与老的还存活的 executors 重新连接。
  cleanup：杀掉所有的老的还存活的 executors 并退出。
  当 slave 不兼容 或 executor 更新时，使用这个选项。
  （默认：reconnect）
  </td>
</tr>
<tr>
  <td>
    --recovery_timeout=VALUE
  </td>
  <td>
  分配给 slave 恢复的时间。如果　slave 恢复所用的时间超过 recovery_timeout，将会被终止。( 默认：15 min )
  </td>
</tr>
<tr>
  <td>
    --registration_backoff_factor=VALUE
  </td>
  <td>
Slave initially picks a random amount of time between <code>[0, b]</code>, where
<code>b = registration_backoff_factor</code>, to (re-)register with a new master.
Subsequent retries are exponentially backed off based on this
interval (e.g., 1st retry uses a random value between <code>[0, b * 2^1]</code>,
2nd retry between <code>[0, b * 2^2]</code>, 3rd retry between <code>[0, b * 2^3]</code>,
etc) up to a maximum of 1mins (default: 1secs)
  </td>
</tr>
<tr>
  <td>
    --resource_estimator=VALUE
  </td>
  <td>
  用于 过度订阅 的 资源评估者 的名称。
  </td>
</tr>
<tr>
  <td>
    --resources=VALUE
  </td>
  <td>
  每个 slave 总的可消耗资源。可以用 JSON 格式提供，也可以是使用分号隔开的 key:value
  键值对列表，配合指定角色选项。
<p/>
key:value 列表:
<code>name(role):value;name:value...</code>
<p/>
To use JSON, pass a JSON-formatted string or use
<code>--resources=filepath</code> to specify the resources via a file containing
a JSON-formatted string. 'filepath' can be of the form
<code>file:///path/to/file</code> or <code>/path/to/file</code>.
<p/>
Example JSON:
<pre><code>[
  {
    "name": "cpus",
    "type": "SCALAR",
    "scalar": {
      "value": 24
    }
  },
  {
    "name": "mem",
    "type": "SCALAR",
    "scalar": {
      "value": 24576
    }
  }
]</code></pre>
  </td>
</tr>
<tr>
  <td>
    --[no-]revocable_cpu_low_priority
  </td>
  <td>
  通过 revocable CPU 以相对低的优先级运行 containers 。 目前只支持 cgroups/cpu isolator 。( 默认: true )
  </td>
</tr>
<tr>
  <td>
    --sandbox_directory=VALUE
  </td>
  <td>
  沙盒被映射到容器中的绝对目录路径。
(默认: /mnt/mesos/sandbox)
  </td>
</tr>
<tr>
  <td>
    --slave_subsystems=VALUE
  </td>
  <td>
  一系列的逗号分隔的 cgroup 子系统来从二进制运行slave。例如，<code>memory,cpuacct</code> 。默认为 none 。此功能用于资源的监视以及 no cgroup 下限制设置，它们从 root mesos cgroup 继承而来。
  </td>
</tr>
<tr>
  <td>
    --[no-]strict
  </td>
  <td>
  如果 <code>strict=true</code>，任何以及所有错误恢复都被认为是致命的。反之，恢复期间，任何预期的错误都会被忽略。 ( 默认： true )
  </td>
</tr>
<tr>
  <td>
    --[no-]switch_user
  </td>
  <td>
  是否用提交它们的用户来运行 tasks 而不是使用运行 slave 的用户. ( 需要 setuid 权限)。 ( 默认: true )
If set to <code>true</code>, the slave will attempt to run tasks as
the <code>user</code> who submitted them (as defined in <code>FrameworkInfo</code>)
(this requires <code>setuid</code> permission and that the given <code>user</code>
exists on the slave).
If the user does not exist, an error occurs and the task will fail.
If set to <code>false</code>, tasks will be run as the same user as the Mesos
slave process.
<b>NOTE</b>: This feature is not yet supported on Windows slave, and
therefore the flag currently does not exist on that platform. (default: true)
  </td>
</tr>
<tr>
  <td>
    --[no-]systemd_enable_support
  </td>
  <td>
  系统支持的最高级控制。当设置为 enabled，像 executor life-time 延期这样的功能都会设置为
  enabled，除非有一个明确的 flag 设置其为 disable。这个会在 agent 作为 systemd unit 发布
  时设置为 enabled。
  （默认：true）
  </td>
</tr>
<tr>
  <td>
    --systemd_runtime_directory=VALUE
  </td>
  <td>
  systemd 系统运行时目录路径。
  （默认：/run/systemd/system）
  </td>
</tr>
<tr>
  <td>
    --work_dir=VALUE
  </td>
  <td>
framework 工作目录的路径。( 默认: /tmp/mesos )
  </td>
</tr>
</table>

##### 当配置了 ' –with-network-isolator ', 以下标记位才会生效：

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
