#Slave 恢复

Slave 恢复是 Mesos 的一个特色功能：

1. 当 slave 进程中断时，Executors/tasks 保持运行。
2. slave 进程重启之后，重新连接到仍在运行 Executors/tasks。

这个功能是 0.14.0 版引入的。当需要升级或者意外崩溃时，Mesos slave 能被重新启动。


#How does it work?
#工作原理

Slave 可以定期将检查点（check point）记录到本地硬盘，检查点的内容包括正在运行的 tasks 和  executors（包括 Task 信息，Executor  信息， 更新状态等）。一旦在 slave 和框架上激活了检查点特性，在他们重启的时候会自动读取既有的检查点信息，重新连接上还在运行的 executor。当然如果之前宿主机整个系统被重启过，那所有 executors/task 都不存在了。

注意：如果要启用 framework 恢复功能，必须先手动激活检查点特性。如果为了避免额外 I/O 开销，也可以手动关闭检查点特性。


#激活 slave 检查点特性

注意：从 Mesos 版本 0.22.0 开始，Slave 会自动激活检查点特性。

作为这个特性的一部分， 在 slave 中添加了四种新的标记。

- `checkpoint`：是否将 slave 和 framework 的检查点信息保存到磁盘。[默认值：true]


- 重启的 slave 恢复状态更新之后，重新连接（`–recover=reconnect`）或者杀掉（`–recover=cleanup`）旧的  executor。
       -  注意： 从  Mesos  版本0.22.0开始， 这个标记也将能够删除所有的 slaves。

- `strict` ：在  strict  模式是否允许恢复。[默认值： true]

 - 如果 strict=true, 在恢复过程中的任何错误都将被认为是致命的。

 - 如果 strict=false,  在恢复期间的任何错误（比如：在检查点数据）都将被忽视， 以便尽可能多地恢复状态。

- `recover`： 是否要恢复状态更新，以及重新连接旧的 executors。 [默认值：reconnect]

 -     如果 recover=reconnect , 重新连接任何存活的 executor。

   - 如果  recover=cleanup, 杀掉所有还存活的 executor，然后退出。这个功能用于 slave 或者  executor 大版本升级之前的环境清理工作。


      - NOTE: If no checkpointing information exists, no recovery is performed and the slave registers with the master as a new slave.


      - 注意：如果不存在检查点信息, slave 重启的时候不会进行恢复，而是和 master 注册为一个新的 slave。


- `recovery_timeout` ： 给 slave 恢复所分配时间。[默认值：15 mins]

   - 如果 slave 重启后用于恢复的时间超过了 `recovery_timeout` 限制, 所有正在等待连接 slave 的 executor 将自行销毁。
      - 注意：这个标记仅适用于已经设置了 `--checkpoint` 时。

NOTE: If none of the frameworks have enabled checkpointing, executors/tasks of frameworks die when the slave dies and are not recovered.

注意：如果所有的框架都没有设置检查点，当 slave 挂掉并且重启无法恢复时，框架的 executor/task 也会挂掉。

一个被重启的  slave  应该在一定时限内重新在 master 注册（目前是75秒）。如果这个 slave 重新注册的间隔超过了这个时限， master  将杀掉这个 slave，进而杀掉这个 slave 上全部还在运行的 executor/task。因此，强烈推荐将 slave 重启这个操作变成自动触发（例如使用 monit 命令来监控并重启服务）。

要查询 Slave 启动选项的完整列表，请在命令行运行  `./mesos-slave.sh –help`

#激活 framework 检查点特性
作为这个特性的一部分，更新了的 `FrameworkInfo` 选项包括一个可选的 `checkpoint` 字段。当 framework 想激活检查点特性，应该在注册到 master 之前设置 `FrameworkInfo.checkpoint=True`。

- 注意: 已经启用检查点的 frameworks 将只能从同样启用了检查点的 slave 节点得到资源 offer。所以在 frameworkInfo 中设置  `checkpoint=True` 之前，请确定集群中有激活了检查点特性的 slave！如果不存在（激活此特性的） slave，那么这个 framework 将得不到任何资源 offer，就不能发布任何 task/executor!

#Upgrading to 0.14.0
#升级到 0.14.0

为了体验 slave 恢复特性，你希望将一个正在运行的 Mesos 集群升级到 0.14.0，请查阅 [upgrade instructions] (http://mesos.apache.org/documentation/latest/upgrades/).
