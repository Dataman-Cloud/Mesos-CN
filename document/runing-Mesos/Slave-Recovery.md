#Slave 恢复

Slave 恢复是 Mesos 的一个特色功能：

1. 当 slave 进程中断时，Executors/tasks 保持运行。
2. slave 进程重启之后，重新连接到仍在运行 Executors/tasks。

这个功能是 0.14.0 版引入的。当需要升级或者意外崩溃时，Mesos slave 能被重新启动。


#How does it work?
#工作原理

Slave recovery works by having the slave checkpoint enough information (e.g., Task Info, Executor Info, Status Updates) about the running tasks and executors to local disk. Once the slave and the framework(s) enable checkpointing, any subsequent slave restarts would recover the checkpointed information and reconnect with the executors. Note that if the host running the slave process is rebooted all the executors/tasks are killed.

Slave 恢复工作是通过读取  slave  检查点检查关于运行任务和  executors  在本地磁盘的信息（比如，Task  信息，Executor  信息， 更新状态）。一旦  slave  和  frameworks  被检查， 随后重启的 slave 能够发现检查点的信息和连接到  executor。注意的是， 如果主机正在运行的  slave  进程重新启动， 其中所有的  executors/tasks  将被杀掉。

NOTE: To enable recovery the framework should explicitly request checkpointing. Alternatively, a framework that doesn’t want the disk i/o overhead of checkpointing can opt out of checkpointing.

注意：要使得framework恢复应该请求检查点。另外，不希望本地磁盘的开销


#启用 slave 检查点功能

注意：从 Mesos 版本 0.22.0 开始，Slave 会自动启动检查点功能。


As part of this feature, 4 new flags were added to the slave.

作为这个特性的一部分， 4种新的标记被添加到  slave  中。




- `checkpoint` : Whether to checkpoint slave and frameworks information to disk [Default: true].



- `checkpoint`：是否检查点  slave  和  framework  信息到磁盘中。[默认值： true]




 - This enables a restarted slave to recover status updates and reconnect with (–recover=reconnect) or kill (–recover=cleanup) old executors.
NOTE: From Mesos 0.22.0 this flag will be removed as it will be enabled for all slaves.
   


 - 这使得一个重启的  slave  能恢复状态更新和重新连接（`–recover=reconnect`）或者杀掉（`–recover=cleanup`）旧的  executors。
     
 

       -  NOTE: From Mesos 0.22.0 this flag will be removed as it will be enabled for all slaves.

       -  注意： 从  Mesos  版本0.22.0开始， 这个标记也将能够删除所有的 slaves。




- `strict` : Whether to do recovery in strict mode [Default: true].

 

- `strict` ：在  strict  模式是否能够恢复。[默认值： true]



 - If strict=true, any and all recovery errors are considered fatal.


 - 如果  strict=true  , 任何和所有恢复错误被认为是致命的。

 - If strict=false, any errors (e.g., corruption in checkpointed data) during recovery are ignored and as much state as possible is recovered.

 - 如果 strict=false,  在恢复期间一些错误（比如：在检查点数据）被忽视， 尽可能多的状态恢复。



- `recover` : Whether to recover status updates and reconnect with old executors[Default: reconnect]. 
- `recover`： 是否要恢复状态更新和重新连接旧的  executors 。 [默认值： reconnect]



  -   If recover=reconnect, Reconnect with any old live executors.
   
 -     如果 recover=reconnect , 重新连接任何旧的存活的 executors。

  - If recover=cleanup, Kill any old live executors and exit. Use this option when doing an incompatible slave or executor upgrade!).

   - 如果  recover=cleanup, 杀掉或者退出存活旧的  executors 。  当执行一个不兼容的  slave  或者  executor  升级时用这个选项。




      - NOTE: If no checkpointing information exists, no recovery is performed and the slave registers with the master as a new slave.

      

      - 注意：  如果没有检查点信息存在,  不执行恢复和注册  master  的  slave  作为一个新的  slave。



- `recovery_timeout` : Amount of time allotted for the slave to recover [Default: 15 mins].



- `recovery_timeout` ： 给  slave  恢复分配时间。[默认值： 15 mins]


 
  - If the slave takes longer than recovery_timeout to recover, any executors that are waiting to reconnect to the slave will self-terminate.

   

   - 如果 slave 恢复的时间比`recovery_timeout`长,  正在等待连接到  slave 的任何  executors  将自己终止。



      - NOTE: This flag is only applicable when `--checkpoint` is enabled.
       

      - 注意：这个标记仅适用在`--checkpoint`是  enable  的时。


NOTE: If none of the frameworks have enabled checkpointing, executors/tasks of frameworks die when the slave dies and are not recovered.

注意：如果一个框架没有检查点， 当  slave  挂掉和没有恢复时，框架的executors/tasks也挂掉。


A restarted slave should re-register with master within a timeout (currently, 75s). If the slave takes longer than this timeout to re-register, the master shuts down the slave, which in turn shuts down any live executors/tasks. Therefore, it is highly recommended to automate the process of restarting a slave (e.g, using monit).

一个重新开始的  slave  应该重新注册  master（目前是75秒）。如果这个  slave  花费在注册时时间比这个时间长， master  将杀掉  这个  slave， 进而关闭任何存活的  executors/tasks。因此， 强烈推荐自动化重新启动一个  slave  的进程。

For the complete list of slave options: `./mesos-slave.sh –help`

slave  选项的完整列表： `./mesos-slave.sh –help`

#Enabling framework checkpointing
# framework 检查点

As part of this feature, `FrameworkInfo` has been updated to include an optional `checkpoint` field. A framework that would like to opt in to checkpointing should set `FrameworkInfo.checkpoint=True` before registering with the master.

作为此功能的一部分，`FrameworkInfo`  已经更新包括一个可选的  `checkpoint`  的字段。一个framework想选择检查点，应该设置  `FrameworkInfo.checkpoint=True`  在 注册  master  之前。



- NOTE: Frameworks that have enabled checkpointing will only get offers from checkpointing slaves. So, before setting `checkpoint=True` on FrameworkInfo, ensure that there are slaves in your cluster that have enabled checkpointing. Because, if there are no checkpointing slaves, the framework would not get any offers and hence cannot launch any tasks/executors!



- 注意: 已经启用检查点的  Frameworks 将只能从 检查点的 slave  得到  offer 。 所以， 在  FrameworkInfo设置  `checkpoint=True`  之前， 确定在你的集群中有检查点的  slave。因为， 如果没有检查点的  slaves， 这个  framework 将得不到任何  offer， 不能发布任何  tasks/executors!

#Upgrading to 0.14.0
#升级到 0.14.0

if you want to upgrade a running Mesos cluster to 0.14.0 to take advantage of slave recovery please follow the [upgrade instructions](http://mesos.apache.org/documentation/latest/upgrades/).

如果你希望利用  slave 恢复升级一个正在运行的 Mesos 集群到0.14.0 ，请查阅[upgrade instructions](http://mesos.apache.org/documentation/latest/upgrades/).
