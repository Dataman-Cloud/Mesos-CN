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
        Explanation
      </th>
    </tr>
  </thead>
<tr>
  <td>
    --appc_store_dir=VALUE
  </td>
  <td>
Directory the appc provisioner will store images in.
(default: /tmp/mesos/store/appc)
  </td>
</tr>
<tr>
  <td>
    --attributes=VALUE
  </td>
  <td>
Attributes of the slave machine, in the form:
<code>rack:2</code> or <code>rack:2;u:1</code>
  </td>
</tr>
<tr>
  <td>
    --authenticatee=VALUE
  </td>
  <td>
Authenticatee implementation to use when authenticating against the
master. Use the default <code>crammd5</code>, or
load an alternate authenticatee module using <code>--modules</code>. (default: crammd5)
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_cpu_enable_pids_and_tids_count
  </td>
  <td>
Cgroups feature flag to enable counting of processes and threads
inside a container. (default: false)
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_enable_cfs
  </td>
  <td>
Cgroups feature flag to enable hard limits on CPU resources
via the CFS bandwidth limiting subfeature. (default: false)
  </td>
</tr>
<tr>
  <td>
    --cgroups_hierarchy=VALUE
  </td>
  <td>
The path to the cgroups hierarchy root. (default: /sys/fs/cgroup)
  </td>
</tr>
<tr>
  <td>
    --[no]-cgroups_limit_swap
  </td>
  <td>
Cgroups feature flag to enable memory limits on both memory and
swap instead of just memory. (default: false)
  </td>
</tr>
<tr>
  <td>
    --cgroups_net_cls_primary_handle
  </td>
  <td>
A non-zero, 16-bit handle of the form `0xAAAA`. This will be used as
the primary handle for the net_cls cgroup.
  </td>
</tr>
<tr>
  <td>
    --cgroups_net_cls_secondary_handles
  </td>
  <td>
A range of the form 0xAAAA,0xBBBB, specifying the valid secondary
handles that can be used with the primary handle. This will take
effect only when the <code>--cgroups_net_cls_primary_handle</code> is set.
  </td>
</tr>
<tr>
  <td>
    --cgroups_root=VALUE
  </td>
  <td>
Name of the root cgroup. (default: mesos)
  </td>
</tr>
<tr>
  <td>
    --container_disk_watch_interval=VALUE
  </td>
  <td>
The interval between disk quota checks for containers. This flag is
used for the <code>posix/disk</code> isolator. (default: 15secs)
  </td>
</tr>
<tr>
  <td>
    --container_logger=VALUE
  </td>
  <td>
The name of the container logger to use for logging container
(i.e., executor and task) stdout and stderr. The default
container logger writes to <code>stdout</code> and <code>stderr</code> files
in the sandbox directory.
  </td>
</tr>
<tr>
  <td>
    --containerizer_path=VALUE
  </td>
  <td>
The path to the external containerizer executable used when
external isolation is activated (<code>--isolation=external</code>).
  </td>
</tr>
<tr>
  <td>
    --containerizers=VALUE
  </td>
  <td>
Comma-separated list of containerizer implementations
to compose in order to provide containerization.
Available options are <code>mesos</code>, <code>external</code>, and
<code>docker</code> (on Linux). The order the containerizers
are specified is the order they are tried.
(default: mesos)
  </td>
</tr>
<tr>
  <td>
    --credential=VALUE
  </td>
  <td>
Either a path to a text with a single line
containing <code>principal</code> and <code>secret</code> separated by whitespace.
Or a path containing the JSON-formatted information used for one credential.
Path could be of the form <code>file:///path/to/file</code> or <code>/path/to/file</code>.
Example:
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
The default container image to use if not specified by a task,
when using external containerizer.
  </td>
</tr>
<tr>
  <td>
    --default_container_info=VALUE
  </td>
  <td>
JSON-formatted ContainerInfo that will be included into
any ExecutorInfo that does not specify a ContainerInfo.
<p/>
See the ContainerInfo protobuf in mesos.proto for
the expected format.
<p/>
Example:
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
Any resources in the <code>--resources</code> flag that
omit a role, as well as any resources that
are not present in <code>--resources</code> but that are
automatically detected, will be assigned to
this role. (default: * )
  </td>
</tr>
<tr>
  <td>
    --disk_watch_interval=VALUE
  </td>
  <td>
Periodic time interval (e.g., 10secs, 2mins, etc)
to check the overall disk usage managed by the slave.
This drives the garbage collection of archived
information and sandboxes. (default: 1mins)
  </td>
</tr>
<tr>
  <td>
    --docker=VALUE
  </td>
  <td>
The absolute path to the docker executable for docker
containerizer.
(default: docker)
  </td>
</tr>
<tr>
  <td>
    --[no-]docker_kill_orphans
  </td>
  <td>
Enable docker containerizer to kill orphaned containers.
You should consider setting this to false when you launch multiple
slaves in the same OS, to avoid one of the DockerContainerizer
removing docker tasks launched by other slaves.
(default: true)
  </td>
</tr>
<tr>
  <td>
    --docker_mesos_image=VALUE
  </td>
  <td>
The docker image used to launch this mesos slave instance.
If an image is specified, the docker containerizer assumes the slave
is running in a docker container, and launches executors with
docker containers in order to recover them when the slave restarts and
recovers.
  </td>
</tr>
<tr>
  <td>
    --docker_registry=VALUE
  </td>
  <td>
The default url for pulling Docker images. It could either be a Docker
registry server url (i.e: <code>https://registry.docker.io</code>), or a local
path (i.e: <code>/tmp/docker/images</code>) in which Docker image archives
(result of <code>docker save</code>) are stored. (default: https://registry-1.docker.io)
  </td>
</tr>
<tr>
  <td>
    --docker_remove_delay=VALUE
  </td>
  <td>
The amount of time to wait before removing docker containers
(e.g., <code>3days</code>, <code>2weeks</code>, etc).
(default: 6hrs)
  </td>
</tr>
<tr>
  <td>
    --docker_socket=VALUE
  </td>
  <td>
The UNIX socket path to be mounted into the docker executor container
to provide docker CLI access to the docker daemon. This must be the
path used by the slave's docker image.
(default: /var/run/docker.sock)
  </td>
</tr>
<tr>
  <td>
    --docker_stop_timeout=VALUE
  </td>
  <td>
The time as a duration for docker to wait after stopping an instance
before it kills that instance. (default: 0ns)
  </td>
</tr>
<tr>
  <td>
    --docker_store_dir=VALUE
  </td>
  <td>
Directory the Docker provisioner will store images in (default: /tmp/mesos/store/docker)
  </td>
</tr>
<tr>
  <td>
    --[no-]enforce_container_disk_quota
  </td>
  <td>
Whether to enable disk quota enforcement for containers. This flag
is used for the <code>posix/disk</code> isolator. (default: false)
  </td>
</tr>
<tr>
  <td>
    --executor_environment_variables=VALUE
  </td>
  <td>
JSON object representing the environment variables that should be
passed to the executor, and thus subsequently task(s). By default the
executor will inherit the slave's environment variables.
Example:
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
Amount of time to wait for an executor
to register with the slave before considering it hung and
shutting it down (e.g., 60secs, 3mins, etc) (default: 1mins)
  </td>
</tr>
<tr>
  <td>
    --executor_shutdown_grace_period=VALUE
  </td>
  <td>
Amount of time to wait for an executor
to shut down (e.g., 60secs, 3mins, etc) (default: 5secs)
  </td>
</tr>
<tr>
  <td>
    --fetcher_cache_dir=VALUE
  </td>
  <td>
Parent directory for fetcher cache directories
(one subdirectory per slave). (default: /tmp/mesos/fetch)
  </td>
</tr>
<tr>
  <td>
    --fetcher_cache_size=VALUE
  </td>
  <td>
Size of the fetcher cache in Bytes. (default: 2GB)
  </td>
</tr>
<tr>
  <td>
    --frameworks_home=VALUE
  </td>
  <td>
Directory path prepended to relative executor URIs (default: )
  </td>
</tr>
<tr>
  <td>
    --gc_delay=VALUE
  </td>
  <td>
Maximum amount of time to wait before cleaning up
executor directories (e.g., 3days, 2weeks, etc).
Note that this delay may be shorter depending on
the available disk usage. (default: 1weeks)
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
Path to find Hadoop installed (for
fetching framework executors from HDFS)
(no default, look for <code>HADOOP_HOME</code> in
environment or find hadoop on <code>PATH</code>) (default: )
  </td>
</tr>
<tr>
  <td>
    --image_providers=VALUE
  </td>
  <td>
Comma-separated list of supported image providers,
e.g., <code>APPC,DOCKER</code>.
  </td>
</tr>
<tr>
  <td>
    --image_provisioner_backend=VALUE
  </td>
  <td>
Strategy for provisioning container rootfs from images,
e.g., <code>bind</code>, <code>copy</code>. (default: copy)
  </td>
</tr>
<tr>
  <td>
    --isolation=VALUE
  </td>
  <td>
Isolation mechanisms to use, e.g., <code>posix/cpu,posix/mem</code>, or
<code>cgroups/cpu,cgroups/mem</code>, or network/port_mapping
(configure with flag: <code>--with-network-isolator</code> to enable),
or <code>external</code>, or load an alternate isolator module using
the <code>--modules</code> flag. Note that this flag is only relevant
for the Mesos Containerizer. (default: posix/cpu,posix/mem)
  </td>
</tr>
<tr>
  <td>
    --launcher=VALUE
  </td>
  <td>
The launcher to be used for Mesos containerizer. It could either be
<code>linux</code> or <code>posix</code>. The Linux launcher is required for cgroups
isolation and for any isolators that require Linux namespaces such as
network, pid, etc. If unspecified, the slave will choose the Linux
launcher if it's running as root on Linux.
  </td>
</tr>
<tr>
  <td>
    --launcher_dir=VALUE
  </td>
  <td>
Directory path of Mesos binaries. Mesos would find health-check,
fetcher, containerizer and executor binary files under this
directory. (default: /usr/local/libexec/mesos)
  </td>
</tr>
<tr>
  <td>
    --oversubscribed_resources_interval=VALUE
  </td>
  <td>
The slave periodically updates the master with the current estimation
about the total amount of oversubscribed resources that are allocated
and available. The interval between updates is controlled by this flag.
(default: 15secs)
  </td>
</tr>
<tr>
  <td>
    --perf_duration=VALUE
  </td>
  <td>
Duration of a perf stat sample. The duration must be less
than the <code>perf_interval</code>. (default: 10secs)
  </td>
</tr>
<tr>
  <td>
    --perf_events=VALUE
  </td>
  <td>
List of command-separated perf events to sample for each container
when using the perf_event isolator. Default is none.
Run command <code>perf list</code> to see all events. Event names are
sanitized by downcasing and replacing hyphens with underscores
when reported in the PerfStatistics protobuf, e.g., <code>cpu-cycles</code>
becomes <code>cpu_cycles</code>; see the PerfStatistics protobuf for all names.
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
The name of the QoS Controller to use for oversubscription.
  </td>
</tr>
<tr>
  <td>
    --qos_correction_interval_min=VALUE
  </td>
  <td>
The slave polls and carries out QoS corrections from the QoS
Controller based on its observed performance of running tasks.
The smallest interval between these corrections is controlled by
this flag. (default: 0secs)
  </td>
</tr>
<tr>
  <td>
    --recover=VALUE
  </td>
  <td>
Whether to recover status updates and reconnect with old executors.
Valid values for <code>recover</code> are
reconnect: Reconnect with any old live executors.
cleanup  : Kill any old live executors and exit.
           Use this option when doing an incompatible slave
           or executor upgrade!). (default: reconnect)
  </td>
</tr>
<tr>
  <td>
    --recovery_timeout=VALUE
  </td>
  <td>
Amount of time allotted for the slave to recover. If the slave takes
longer than recovery_timeout to recover, any executors that are
waiting to reconnect to the slave will self-terminate.
(default: 15mins)
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
The name of the resource estimator to use for oversubscription.
  </td>
</tr>
<tr>
  <td>
    --resources=VALUE
  </td>
  <td>
Total consumable resources per slave. Can be provided in JSON format
or as a semicolon-delimited list of key:value pairs, with the role
optionally specified.
<p/>
As a key:value list:
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
Run containers with revocable CPU at a lower priority than
normal containers (non-revocable cpu). Currently only
supported by the cgroups/cpu isolator. (default: true)
  </td>
</tr>
<tr>
  <td>
    --sandbox_directory=VALUE
  </td>
  <td>
The absolute path for the directory in the container where the
sandbox is mapped to.
(default: /mnt/mesos/sandbox)
  </td>
</tr>
<tr>
  <td>
    --slave_subsystems=VALUE
  </td>
  <td>
List of comma-separated cgroup subsystems to run the slave binary
in, e.g., <code>memory,cpuacct</code>. The default is none.
Present functionality is intended for resource monitoring and
no cgroup limits are set, they are inherited from the root mesos
cgroup.
  </td>
</tr>
<tr>
  <td>
    --[no-]strict
  </td>
  <td>
If <code>strict=true</code>, any and all recovery errors are considered fatal.
If <code>strict=false</code>, any expected errors (e.g., slave cannot recover
information about an executor, because the slave died right before
the executor registered.) during recovery are ignored and as much
state as possible is recovered.
(default: true)
  </td>
</tr>
<tr>
  <td>
    --[no-]switch_user
  </td>
  <td>
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
Top level control of systemd support. When enabled, features such as
executor life-time extension are enabled unless there is an explicit
flag to disable these (see other flags). This should be enabled when
the agent is launched as a systemd unit.
(default: true)
  </td>
</tr>
<tr>
  <td>
    --systemd_runtime_directory=VALUE
  </td>
  <td>
The path to the systemd system run time directory.
(default: /run/systemd/system)
  </td>
</tr>
<tr>
  <td>
    --work_dir=VALUE
  </td>
  <td>
Directory path to place framework work directories
(default: /tmp/mesos)
  </td>
</tr>
</table>








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
