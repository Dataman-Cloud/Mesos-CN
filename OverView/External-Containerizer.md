###External Containerizer
- EC = external containerizer 。 mesos slave 的一部分，提供 API ，通过可执行的外部插件来支持 containerizing ( 集装箱化 )。
- ECP = external containerizer 程序。 一个通过 containering system 接口实现了真实的集装箱化的可执行外部插件 。 ( 例如 Docker )
 
###Containerizing ( 集装箱化 )

####General Overview（ 总体概述 ）
EC 调用 ECP 作为一个 shell 进程，将 shell 中的命令作为参数传递给 ECP 执行 。 额外的数据则通过 stdin 和 stdout 传递。


ECP 会对所有它能够处理的命令返回一个 " 0 " ,非 " 0 " 状态码则表示发出的错误信号。下面你会找到必须在 ECP 上实现的命令的概述，以及它们的调用机制。

ECP 有望使用 stderr 来显示更多的调试信息和状态信息。这些信息记录到一个文件中。

####Call and communication scheme ( 调用与通信方案 )

接口描述了一个 ECP 具有通过命令调用来实现的功能。在 ECP 上的许多调用还将 protobuf 消息通过 stdin 传递。 在 ECP ，有些调用期望通过 strout 提供一个 protobuf 消息返回。

#####COMMAND < INPUT-PROTO > RESULT-PROTO

* ***launch < containerizer::Launch***

* ***update < containerizer::Update***

* ***usage < containerizer::Usage > mesos::ResourceStatistics***

* ***wait < containerizer::Wait > containerizer::Termination***

* ***destroy < containerizer::Destroy***

* ***containers > containerizer::Containers***

* ***recover***

###Command Ordering (命令顺序)

####不要擅自假设

命令几乎可以以任意顺序执行，只有一个例外。当启动一个 task 时， EC 将确保 ECP 第一个接收到指定的 container 。所有的命令都需要排在后边，直到 ECP 返回启动。

###用例

####Task 启动 EC / ECP 概述
* EC 在 ECP 上调用 ***launch*** 命令。

* 随着调用， ECP 会通过 stdin 接收到一个 " containerizer::Launch "  protobuf 消息。

* ECP 确保 executor 已经启动。注意：启动是不应该被阻塞的。它应该触发 executor / command 后立即返回 ，这可以通过 ECP 中的  fork-exec 完成。

* EC 在 ECP 上调用 ***wait*** 命令。

* 随着调用，ECP 会通过 stdin 接受到一个 " containerizer::Wait "  protobuf 消息。

* ECP 现在阻塞，直到收到启动命令。启动命令可以通过 ECP 中的 waitpid 实现。

* 一但收到命令， ECP 应该通过 strout 提供一个 " containerizer::Termination  " protobuf 消息， 返回给 EC 。

###Container 生命周期图

####Container 启动

####Container 运行

####资源限制

###Slave 恢复概述

* Slave 恢复通过 check pointed 状态。

* EC 通过 ECP 调用 ***recover*** ，在这个命令中没有 protobuf 消息发送或者预测结果

* 如果有必要， ECP可以通过自己的故障恢复机制来恢复内部的状态。

* 当 ***recover*** 命令返回， EC 将通过 ECP 执行 *** container***。

* ECP 会返回一个当前处于 active 状态的 cantainers 列表。注意：这些 container 应该是被 ECP 所知的。但实际情况下部分是未知的 slave (例如 slave 启动后失败或者正在等待)。这些 container 被认为是孤儿，

* EC 会把 slave 已知的列表中 containers 与 ***Containers*** 命令列出的进行比较。对于每个被标记为孤儿的 container， slave 将在 ECP 上调用 ***wait*** 命令后，再调用 ***destroy***  销毁它们。

* slave 在 ECP 上调用（通过 EC） ***wait*** 命令来恢复所有 containers。This does once again put wait into the position of the ultimate command reaper.


###Slave 恢复流程图

####Recovery

####Orphan Destruction( 孤儿的销毁 )

###Command 细节

####launch

####启动 containerized 执行器

手中所有的 ECP 信息需要通过一个 executor 来启动一个 task。这个调用不会等待 executor / command 返回。 containeried 的实际结果命令通　***wait*** 调用。
```
launch < containerizer::Launch
```

这个调用通过 stdin 接收 containerizer::Launch protobuf

```
/**
 * Encodes the launch command sent to the external containerizer
 * program.
 */
message Launch {
  required ContainerID container_id = 1;
  optional TaskInfo task_info = 2;
  optional ExecutorInfo executor_info = 3;
  optional string directory = 4;
  optional string user = 5;
  optional SlaveID slave_id = 6;
  optional string slave_pid = 7;
  optional bool checkpoint = 8;
}
```

这个调用不会从 stdout 返回任何数据。


####wait

####容器的executor终止时获取信息
期待去获得 executor / command 。这个调用会阻塞，直到 executor / command 终止。
```
wait < containerizer::Wait > containerizer::Termination
```

这个调用通过 stdin 接收  containerizer::Wait protobuf 。
```
/**
 * Encodes the wait command sent to the external containerizer
 * program.
 */
message Wait {
  required ContainerID container_id = 1;
}
```

这个调用期望通过 stdout 返回 containerizer::Termination。

```
/**
 * Information about a container termination, returned by the
 * containerizer to the slave.
 */
 message Termination {
  // A container may be killed if it exceeds its resources; this will
  // be indicated by killed=true and described by the message string.
  required bool killed = 1;
  required string message = 2;

  // Exit status of the process.
  optional int32 status = 3;
}
```
Termination 参数 ***killed*** 只有当 containerizer 或底层隔离必须通过杀死任务来实现限制时才设置。

####update

####更新 container 的资源限制。

发送 ( 新的 ) 资源限制给 container 。资源限制在 container 可能会改变 container 任务的生命周期

```
update < containerizer::Update
```

这个调用通过 stdin 接收 containerizer::Update
```
/**
 * Encodes the update command sent to the external containerizer
 * program.
 */
message Update {
  required ContainerID container_id = 1;
  repeated Resource resources = 2;
}
```
这个调用不会通过 stdout 返回任何数据。

####usage

####一个 Container 任务收集使用情况

用于调查当前容器的资源使用情况

```
usage < containerizer::Usage > mesos::ResourceStatistics
```

这个调用通过 stdin 接收 containerizer::Usage protobuf 。
```
/**
 * Encodes the usage command sent to the external containerizer
 * program.
 */
message Usage {
  required ContainerID container_id = 1;
}
```

这个调用期望通过 stdout 返回 mesos::ResourceStatistics 。
```
/*
 * A snapshot of resource usage statistics.
 */
message ResourceStatistics {
  required double timestamp = 1; // Snapshot time, in seconds since the Epoch.

  // CPU Usage Information:
  // Total CPU time spent in user mode, and kernel mode.
  optional double cpus_user_time_secs = 2;
  optional double cpus_system_time_secs = 3;

  // Number of CPUs allocated.
  optional double cpus_limit = 4;

  // cpu.stat on process throttling (for contention issues).
  optional uint32 cpus_nr_periods = 7;
  optional uint32 cpus_nr_throttled = 8;
  optional double cpus_throttled_time_secs = 9;

  // Memory Usage Information:
  optional uint64 mem_rss_bytes = 5; // Resident Set Size.

  // Amount of memory resources allocated.
  optional uint64 mem_limit_bytes = 6;

  // Broken out memory usage information (files, anonymous, and mmaped files)
  optional uint64 mem_file_bytes = 10;
  optional uint64 mem_anon_bytes = 11;
  optional uint64 mem_mapped_file_bytes = 12;
}
```


####destroy

####终止 container 的 executor


在比较少见的情况下才被使用。像 slave 关闭，但 slave 又在故障转移。
```
destroy < containerizer::Destroy
```
这个调用通过 stdin 接收 containerizer::Destroy protobuf 。

```
/**
 * Encodes the destroy command sent to the external containerizer
 * program.
 */
message Destroy {
  required ContainerID container_id = 1;
}
```
这个调用不返回任何数据。

####containers

####获取所有 active 状态的 container ID
返回所有目前 active 状态的 container 标识符。
```
containers > containerizer::Containers
```
这个调用不通过 stdin 接收任何额外的数据。

此调用有望通过 stdout ，再经过 " containerizer::Containers "返回。 

```
/**
 * Information on all active containers returned by the containerizer
 * to the slave.
 */
message Containers {
  repeated ContainerID containers = 1;
}
```

####recover

####ECP 内部状态恢复

允许 ECP 对自己的状态进行恢复。如果 ECP 使用 check-pointing 定期保存状态，例如通过文件系统。则此调用将会是很好的时机来反序列化状态信息来进行恢复。
```
recover
```

此调用不会通过 stdin 接收任何额外数据。也不通过 stdout 返回数据。


####Protobuf Message 定义

可能更多上述提及的 protobufs 在新版本的 Protobuf Message 中，和  protobuf messages 引用它们的方式一样。请检查：

* containerizer::XXX 在 include/mesos/containerizer/containerizer.proto 中定义。
* mesos::XXX 在 include/mesos/mesos.proto 中定义。

### Environment ( 环境 )

####Sandbox（ 沙盒 ）

一个 sandbox 环境由 ***cd*** 命令进入到 executor 的工作目录以及 stderr 重定向到 executor 的 " strerr " log 目录。注意：不是所有的 invocations 都有一个完整的 sandbox 环境。

###Addional Environment Variables

此外，当调用 ECP 的时候，要设置一些新的环境变量。

* MESOS_LIBEXEC_DIRECTORY = path to mesos-executor, mesos-usage, …  这些信息总是存在的。

* MESOS_WORK_DIRECTORY = slave work directory. 此变量被用来区分 slave 实例。这些信息同样也总是存在的。注意，将一组容器打包到一个 slave 实例，从而在需要的时候可以恢复。这点是很有帮助性的。

* MESOS_DEFAULT_CONTAINER_IMAGE = default image as provided via slave flags ( default_container_image )。只是在调用 ***launch*** 时需要被提供。

###Debugging ( 调试 )
####提高日志级别

***GLOG_v=2 ./bin/mesos-slave --master=[...]*** 在 Mesos 启动前，通过调用并设置为大于等于二来增加日记管理级别。

###ECP 标准错误日志

所有输出到你的 ECP 都将记录到 executor 的 " stderr " 日志目录。

日志输出例子：
```
I0603 02:12:34.165662 174215168 external_containerizer.cpp:1083] Invoking external containerizer for method 'launch'
I0603 02:12:34.165675 174215168 external_containerizer.cpp:1100] calling: [/Users/till/Development/mesos-till/build/src/test-containerizer launch]
I0603 02:12:34.165678 175824896 slave.cpp:497] Successfully attached file '/tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345'
I0603 02:12:34.165686 174215168 external_containerizer.cpp:1101] directory: /tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345
```
ECP 将这个 stderr 的路径位置显示在日志的最后一行：

```
cat /tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345/stderr
```