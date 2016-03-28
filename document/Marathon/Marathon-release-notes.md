
# Marathon v1.0.0-RC1 Release Note

## 0.15.3 版本 到 1.0.0 版本的变化

### 建议使用 0.28.0 版本的 Mesos

### 重大修改

#### 任务发布中新的默认设置

Marathon 做了很多设置调整。我们的目标是，让默认值对于中小型集群来说更加合理。
我们意识到，一些默认值可能并不适合一些小规模的场景，所以我们修改了这些值：

* `--launch_tokens` 从1000修改为100
* `--max_tasks_per_offer` 从100修改为5
* `--reconciliation_interval` 从300000（5分钟）修改为600000（10分钟）

#### 更新了验证插件接口

为了支持更加复杂的插件，我们重新设计了认证和授权插件接口。

### 概览

#### 支持持久化存储

现在可以发布支持持久化存储卷的任务了。这些存储卷可以通过使用 UI 界面或 REST API 来指定。
Marathon 会保留匹配的 Agent 上所有必需的资源，随后如果需要，可以将任务发布在同一个 Agent 上。
甚至在重新发布相关联的任务之后，存储卷中的数据任然会被保留。
此版本提供的是基础功能，将来还会继续扩展。所以使用时请自担风险：）

[查看持久化存储卷详情和配置案例>>](https://mesosphere.github.io/marathon/docs/persistent-volumes.html)

#### 支持定制元数据接口

v2版的 REST API 已经可以支持通过`portDefinition`应用域来添加元数据接口（协议，名称，标签）。
Marathon 会将这个信息传递给 Mesos。Mesos 则会将此信息用于服务发现。

注意：`portDefinitions` 数组将会替换 `ports` 数组。

#### 支持 HTTP 插件扩展

插件目前可以实现 HTTP endpoints。

#### 增加了 Leader 持续时间指标

这个指标显示了从上一次Leader选举后到目前为止的时间。
这个指标可以用于诊断稳定性问题和查看 Leader 选举会多久发生一次。

#### 更易懂的错误信息

无论是从人还是计算机的角度来看，API 错误信息现在都变得更加一致和易懂。

#### 大量的文档更新

#### 通过分批kill应用实例，来提高停止和重启任务的体验。

当停止或重启一个应用时，Marathon 将分批 Kill 应用实例。这样可以避免压垮 Mesos。
分批处理的数量和频率可以通过参数来配置。

#### 支持 Mesos 0.28 版本的 TASK_KILLING 状态标识

这使得 Marathon 让 Mesos 通过 `--enable_features task_killing` 标记来使用 `TASK_KILLING`
状态标识。Marathon 不再使用此任务状态标识。

### WEB UI

此版本的 Marathon 在 WEB UI 方面也有了很多改进。

#### 应用和搜索

* 提高了全局搜索能力（模糊识别）
* 组也可以作为搜索结果的一部分来展现了
* 应用列表支持浏览空的组
* 可以通过UI直接创建空的组
* 可以通过一个边栏来筛选过滤匹配的应用

#### 创建和编辑窗口

* 重新设计了窗口提高了易用性
* 全新的 JSON 编辑器
* 可以通过UI创建支持本地持久化存储卷的常驻任务
* 全部 UI 方面的变化请参考：[Marathon UI release pages](https://github.com/mesosphere/marathon-ui/releases)

### 修复的问题

* [#929](https://github.com/mesosphere/marathon/issues/929) - Allow tcp,udp ports in portMappings
* [#2751](https://github.com/mesosphere/marathon/issues/2751) - Commit suicide on ZK exceptions
* [#3091](https://github.com/mesosphere/marathon/issues/3091) - App updates hanging on downscales
* [#3169](https://github.com/mesosphere/marathon/issues/3169) - Possible to start app with negative resources
* [#3241](https://github.com/mesosphere/marathon/issues/3241) - Serverside validation messages are inconsistent
* [#3251](https://github.com/mesosphere/marathon/issues/3251) - Tried to kill an existing app, said it doesn't exist even though it does
* [#3338](https://github.com/mesosphere/marathon/issues/3338) - Path in health checks validation failure results is broken
* [#3367](https://github.com/mesosphere/marathon/issues/3367) - Relative paths for dependencies not working anymore
* [#3377](https://github.com/mesosphere/marathon/issues/3377) - Marathon should remove the FrameworkId for special Mesos errors
* [#3385](https://github.com/mesosphere/marathon/issues/3385) - Creating an empty group using an existing app ID should return 409
* [#3402](https://github.com/mesosphere/marathon/pull/3402) - Race conditions in HttpEventActor
* [#3423](https://github.com/mesosphere/marathon/issues/3423) - Report kills due to failed healthcheck.
* [#3439](https://github.com/mesosphere/marathon/issues/3439) - Relative paths in dependencies should be resolvable.

### 贡献者

|Commits |Contributor |
|--------|:----------:|
|101|Matthias Veit|
|60	|Matthias Eichstedt|
|54	|Peter Kolloch|
|43	|Gastón Kleiman|
|11	|sascala|
|11	|Alexander Weber|
|5	|Pierluigi Cau|
|5	|Suzanne Scala|
|3	|Aaron Bell|
|3	|Sunil Shah|
|3	|alenkacz|
|3	|Joerg Schad|
|2	|Isabel Jimenez|
|2	|Tomasz Janiszewski|
|2	|Pradeep Sekar|
|2	|philipnrmn|
|2	|jlamillan|
|2	|Dr. Stefan Schimanski|
|2	|Timo Reimann|
|2	|Lee Munroe|
|1	|pierrecdn|
|1	|Leonardo Trabuco|
|2	|Lukas Loesche|
|1	|Peter Kelley|
|1	|Brian Antonelli|
|1	|Philip Norman|
|1	|Philipp Hinrichsen|
|1	|Kiril Nesenko|
|1	|Timothée GERMAIN|
|1	|Zhou Weitao|

### 下载：

Tarball:

[http://downloads.mesosphere.com/marathon/v1.0.0-RC1/marathon-1.0.0-RC1.tgz](http://downloads.mesosphere.com/marathon/v1.0.0-RC1/marathon-1.0.0-RC1.tgz)

SHA:

[http://downloads.mesosphere.com/marathon/v1.0.0-RC1/marathon-v1.0.0-RC1.tgz.sha256](http://downloads.mesosphere.com/marathon/v1.0.0-RC1/marathon-v1.0.0-RC1.tgz.sha256)

Docker:

[https://registry.hub.docker.com/u/mesosphere/marathon](https://registry.hub.docker.com/u/mesosphere/marathon) with tag v1.0.0-RC1

#### 源码下载：

[源代码（zip版）](http://github.com/mesosphere/marathon/archive/v1.0.0-RC1.zip)

[源代码（tar.gz版）](http://github.com/mesosphere/marathon/archive/v1.0.0-RC1.tar.gz)


内容编译自 [Marathon v1.0.0-RC1](https://github.com/mesosphere/marathon/releases) By tsingguo
