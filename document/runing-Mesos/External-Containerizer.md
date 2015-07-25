# 外部 containerizer 

#外部 Containerizer

 - EC = 外部 containerizer 。 mesos slave 的一部分其提供 API ，通过可执行的外部插件来支持 containerizing ( 集装箱化 )。 
 - ECP = 外部 containerizer 程序。 一个通过 containering
   system 接口实现了真正的集装箱化的可执行外部插件 。 ( 例如 Docker ) 

#Containerizing

总体概述

EC 调用 ECP 作为一个 shell 进程，将 shell 中的命令作为参数传递给 ECP 执行 。 额外的数据则通过 stdin 和 stdout 传递。

ECP 会对所有它能够处理的命令返回一个 " 0 "状态码 ,非 " 0 " 状态码则表示发出的错误信号。下面你会找到必须在 ECP 上实现的命令的概述，以及它们的调用机制。

ECP 期望能够使用 stderr 来显示更多的调试信息和状态信息。这些信息将被记录到一个文件中。 请参考 [Enviroment: Sandbox][1]

#调用与通信方案

接口描述了一个 ECP 具有通过命令调用来实现的功能。在 ECP 上的许多调用还将 protobuf 消息通过 stdin 在相互间传递。有些 ECP 之上的调用被期望能够通过 strout 提供一个 protobuf 消息返回。所有的 protobuf 消息以他们的原始长度作为前缀 - 这有时被称为"Record-IO"格式。 请参考[Record-IO 序列化/反序列化示例][2]

 #COMMAND < INPUT-PROTO > RESULT-PROTO
   
   
 - launch < containerizer::Launch
 - update < containerizer::Update
 - usage < containerizer::Usage > mesos::ResourceStatistics
 - wait < containerizer::Wait > containerizer::Termination
 -  destroy < containerizer::Destroy
 - containers > containerizer::Containers
 - recover

#命令顺序

#不要擅自假设

命令几乎可以以任意顺序执行，对此只有一个例外： 当启动一个 task 时， EC 将确保 ECP 首先在指定的容器上接收一个 launch 。所有的命令都需要排在后边，直到 luanch 从 ECP 能够返回。

#用例

#Task 启动 EC / ECP 概述

 - EC 在 ECP 上调用 launch 命令。
 - 随着调用， ECP 会通过 stdin 接收到一个 " containerizer::Launch " protobuf 消息。
 - ECP 确保 executor 已经启动。注意：启动是不应该被阻塞的。它应该触发 executor / command 后立即返回 ，这可以通过 ECP 中的 fork-exec 完成。
 - EC 在 ECP 上调用 wait 命令。
 - 随着调用，ECP 会通过 stdin 接受到一个 " containerizer::Wait " protobuf 消息。
 - ECP 现在阻塞，直到再次收到启动命令。启动命令可以通过 ECP 中的 waitpid 实现。
 - 一但收到命令， ECP 应该通过 strout 提供一个 " containerizer::Termination " protobuf 消息， 返回给 EC 。

容器 生命周期图

#容器启动  
一个容器在准备状态，现在开始启动和保留直到其进入一个最终状态。
![容器启动][3]

#容器运行  
一个容器在某个点上得到启用，当下被 slave 认作将进入一个非终端状态，以下的命令在 ECP 将得到多次触发在一个容器的整个生命周期中。其触发序列没有没定义。
![容器运行][4]


#资源限制
当一个容器运行，一个资源限制(如：超过内存容量)将通过 ECP 隔离模式的选择来定义
![资源限制][5]


#Slave 恢复概述

 - 通过监测点状态来恢复 Slave 。
 - EC 通过 ECP 调用 recover ，这里没有 protobuf 消息发送或者如预期的那样从该命令返回一个结果。
 - 如果有必要， ECP可以通过自己的故障恢复机制来恢复内部的状态。
 - 当 recover 命令返回， EC 将通过 ECP 执行 *** container***。
 - ECP 会返回一个当前处于 active 状态的 cantainers 列表。注意：这些 container 应该是被 ECP 所知的。但实际情况下部分是未知的 slave (例如 slave 启动后失败或者正在等待)。这些 container 被认为是孤儿，
 - EC 会把 slave 已知的列表中 containers 与 Containers 命令列出的进行比较。对于每个被标记为孤儿的 container， slave 将在 ECP 上调用 wait 命令后，再调用 destroy 销毁它们。
 - slave 在 ECP 上调用（通过 EC） wait 命令来恢复所有 containers。其将在等待无限的命令收割中再次被执行

#Slave 恢复流程图
#恢复
当容器运行时， slave 将执行故障转移。  
![故障转移][6]
#孤儿的销毁
容器被 ECP 所识别的将被运行但 slave 状态可恢复的能力将被终止。  
![孤儿的销毁][7]

#命令细节

#launch

#启动容器化的执行者

通过一个执行者来传递所有的 ECP 需要启用一个作业需要的信息。这个调用不会等待 执行器/命令返回。 容器化的实际结果命令通过 wait 调用。

    launch < containerizer::Launch
    
这个调用通过 stdin 接收 containerizer::Launch protobuf

    /**
    * 对发送到外部容器化程序的启用指令进行解码。
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
这个调用不会从 stdout 返回任何数据。

#等待

#在容器化的执行者终止时获取到信息

这个调用被期望获得到执行者/命令。这个调用会阻直到执行者/命令被终止。

    wait < containerizer::Wait > containerizer::Termination

这个调用通过 stdin 接收 containerizer::Wait protobuf 。

    /**
    * Encodes the wait command sent to the external containerizer
    * program.
    */
    message Wait {
        required ContainerID container_id = 1;
    }
    
这个调用被期望能够通过 stdout 返回 containerizer::Termination。

    /**
     * 关于一个容器的终止信息, containerizer 到 slave 的返回
     */
     message Termination {
      // 一个容器可能被杀死如果它超过了自己的资源使用上限；其将被 killed=true 所标注并被信息串所描述。
      required bool killed = 1;
      required string message = 2;
      // 进程的退出状态值
      optional int32 status = 3;
    }
    
终止的属性 killed 只有当 containerizer 或底层隔离必须通过杀死任务来实现限制时才被设置(比如：task 超过了建议的内存限制)。

#更新

#更新容器的资源限制。

用来针对给定的容器发送(新的)资源限制指令。在容器化的 task 的整个生命周期中附加给容器的资源限制可能会改变。

    update < containerizer::Update
    
这个调用通过 stdin 接收 containerizer::Update

    /**
    * 解析更新命令并发送给外部 containerizer 程序。
    */
    message Update {
        required ContainerID container_id = 1;
        repeated Resource resources = 2;
    }
    
这个调用不会通过 stdout 返回任何数据。

#使用

#针对一个容器化的任务收集相关使用情况

用于调查当前容器的资源使用情况

usage < containerizer::Usage > mesos::ResourceStatistics
这个调用通过 stdin 接收 containerizer::Usage protobuf 。

    /**
    *解码 usage 命令并发送到外部 containerizer 程序 
    */
    message Usage {
    required ContainerID container_id = 1;
    }
    
这个调用被期望能通过 stdout 返回 mesos::ResourceStatistics 。

    /*
    * 一个资源使用统计的快照。
    */
    message ResourceStatistics {
    required double timestamp = 1; // Snapshot time, in seconds since the Epoch.
    // CPU Usage Information: CPU 使用信息：
    // 用户模式和内核模式下总 CPU 使用时间。
    optional double cpus_user_time_secs = 2;
    optional double cpus_system_time_secs = 3;
    // 多少个 CPU 被分配。
    optional double cpus_limit = 4;
    // cpu.stat 针对进程限制(对于竞争问题)
    optional uint32 cpus_nr_periods = 7;
    optional uint32 cpus_nr_throttled = 8;
    optional double cpus_throttled_time_secs = 9;
    // 内存使用信息:
    optional uint64 mem_rss_bytes = 5; // Resident Set Size.
    // 多少内存资源被分配。
    optional uint64 mem_limit_bytes = 6;
    // 分别的内存使用信息  (files, anonymous, and mmaped files)
    optional uint64 mem_file_bytes = 10;
    optional uint64 mem_anon_bytes = 11;
    optional uint64 mem_mapped_file_bytes = 12;
    }

#销毁

终止容器的执行者 

在比较少见的情况下才被使用。像 slave 安全的关闭，但同时也出现在 slave 故障转移的场景 - 更多信息请参考 Slave 恢复。

    destroy < containerizer::Destroy
    
这个调用通过 stdin 接收 containerizer::Destroy protobuf 。

    /**
    * 解码销毁命令并传递给外部的 containerizer 应用
    */
    message Destroy {
        required ContainerID container_id = 1;
    }
这个调用不返回任何数据。

#容器

#获取所有 active 状态的 container ID
返回所有目前 active 状态的 container 标识符。

    containers > containerizer::Containers
这个调用不通过 stdin 接收任何额外的数据。

此调用有望通过 stdout ，再经过 " containerizer::Containers "返回。

    /**
    * containerizer 将所有活跃的容器信息返回给 slave.
    */
    message Containers {
        repeated ContainerID containers = 1;
    }

#恢复

ECP 内部状态恢复

允许 ECP 对自己的状态进行恢复。如果 ECP 使用 check-pointing 定期保存状态，例如通过文件系统。则此调用将会是很好的时机来反序列化状态信息来进行恢复。更多的信息请参考 [slave 恢复概述][8]

    recover
此调用不会通过 stdin 接收任何额外数据。也不通过 stdout 返回数据。

#Protobuf Message 定义

更多以上提及的 protobufs 和 Protobuf Message 在新版本的引用是否雷同。请检查：

    - containerizer::XXX 在 include/mesos/containerizer/containerizer.proto 中定义。
    - mesos::XXX 在 include/mesos/mesos.proto 中定义。
    
#环境

#沙盒 
一个沙盒环境由 cd 命令进入到执行者的工作目录以及 stderr 重定向到执行者的 " strerr " 日志 目录。注意：不是所有的执行都有一个完整的沙盒环境。

其他的环境变量

此外，当调用 ECP 的时候，要设置一些新的环境变量。

    - MESOS_LIBEXEC_DIRECTORY = path to mesos-executor, mesos-usage, … 这些信息总是存在的。
    - MESOS_WORK_DIRECTORY = slave work directory. 此变量被用来区分 slave 实例。这些信息同样也总是存在的。
注意，将一组容器打包到一个 slave 实例，从而在需要的时候可以恢复。这点是很有帮助性的。
    
    - MESOS_DEFAULT_CONTAINER_IMAGE = 默认的镜像通过 slave 的标志位设置来提供( default_container_image )。只是在调用 launch 时需要被提供。

#调试

#增强的详细日志记录
为了能够从 EC 接收到一个高等级的状态信息，需要使用 GLOG 详细等级.在 Mesos 启动前，通过调用并设置为大于等于二来增加日记管理级别。

    GLOG_v=2 ./bin/mesos-slave --master=[...] 

#ECP 标准错误日志

所有输出到你的 ECP 都将记录到执行者的 " stderr " 日志文件。该特定的目录可以从 EC 的 [增强的详细日志记录][9] 中被提取出来。

日志输出例子：
        
        I0603 02:12:34.165662 174215168 external_containerizer.cpp:1083] Invoking external containerizer for method 'launch'
        I0603 02:12:34.165675 174215168 external_containerizer.cpp:1100] calling: [/Users/till/Development/mesos-till/build/src/test-containerizer launch]
        I0603 02:12:34.165678 175824896 slave.cpp:497] Successfully attached file '/tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345'
        I0603 02:12:34.165686 174215168 external_containerizer.cpp:1101] directory: /tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345
        
ECP  针对该调用的 stderr 输出 可以在路径位置显示在日志的最后的标注行的 stderr 文件中找到：

    cat /tmp/ExternalContainerizerTest_Launch_lP22ci/slaves/20140603-021232-16777343-51377-7591-0/frameworks/20140603-021232-16777343-51377-7591-0000/executors/1/runs/558e0a69-70da-4d71-b4c4-c2820b1d6345/stderr

#附加信息
#Record-IO 协议例子: Launch
这里显示一个完整的 record-io 格式化的 protobuf.
#name: offset

    - length: 00 - 03 = record length in byte
    - payload: 04 - (length + 4) = protobuf payload

长度样例: 00000240h = 576 byte 整个 protobuf 大小

十六进制格式样例 

    00000000:  4002 0000 0a26 0a24 3433 3532 3533 6162 2d64 3234 362d 3437  :@....&.$435253ab-d246-47
    00000018:  6265 2d61 3335 302d 3335 3432 3034 3635 6438 3638 1a81 020a  :be-a350-35420465d868....
    00000030:  030a 0131 2a16 0a04 6370 7573 1000 1a09 0900 0000 0000 0000  :...1*...cpus............
    00000048:  4032 012a 2a15 0a03 6d65 6d10 001a 0909 0000 0000 0000 9040  :@2.**...mem............@
    00000060:  3201 2a2a 160a 0464 6973 6b10 001a 0909 0000 0000 0000 9040  :2.**...disk............@
    00000078:  3201 2a2a 180a 0570 6f72 7473 1001 220a 0a08 0898 f201 1080  :2.**...ports..".........
    00000090:  fa01 3201 2a3a 2a1a 2865 6368 6f20 274e 6f20 7375 6368 2066  :..2.*:*.(echo 'No such f
    000000a8:  696c 6520 6f72 2064 6972 6563 746f 7279 273b 2065 7869 7420  :ile or directory'; exit
    000000c0:  3142 2b0a 2932 3031 3430 3532 362d 3031 3530 3036 2d31 3637  :1B+.)20140526-015006-167
    000000d8:  3737 3334 332d 3535 3430 332d 3632 3536 372d 3030 3030 4a3d  :77343-55403-62567-0000J=
    000000f0:  436f 6d6d 616e 6420 4578 6563 7574 6f72 2028 5461 736b 3a20  :Command Executor (Task:
    00000108:  3129 2028 436f 6d6d 616e 643a 2073 6820 2d63 2027 7768 696c  :1) (Command: sh -c 'whil
    00000120:  6520 7472 7565 203b 2e2e 2e27 2952 0131 22c5 012f 746d 702f  :e true ;...')R.1"../tmp/
    00000138:  4578 7465 726e 616c 436f 6e74 6169 6e65 7269 7a65 7254 6573  :ExternalContainerizerTes
    00000150:  745f 4c61 756e 6368 5f6c 5855 6839 662f 736c 6176 6573 2f32  :t_Launch_lXUh9f/slaves/2
    00000168:  3031 3430 3532 362d 3031 3530 3036 2d31 3637 3737 3334 332d  :0140526-015006-16777343-
    00000180:  3535 3430 332d 3632 3536 372d 302f 6672 616d 6577 6f72 6b73  :55403-62567-0/frameworks
    00000198:  2f32 3031 3430 3532 362d 3031 3530 3036 2d31 3637 3737 3334  :/20140526-015006-1677734
    000001b0:  332d 3535 3430 332d 3632 3536 372d 3030 3030 2f65 7865 6375  :3-55403-62567-0000/execu
    000001c8:  746f 7273 2f31 2f72 756e 732f 3433 3532 3533 6162 2d64 3234  :tors/1/runs/435253ab-d24
    000001e0:  362d 3437 6265 2d61 3335 302d 3335 3432 3034 3635 6438 3638  :6-47be-a350-35420465d868
    000001f8:  2a04 7469 6c6c 3228 0a26 3230 3134 3035 3236 2d30 3135 3030  :*.till2(.&20140526-01500
    00000210:  362d 3136 3737 3733 3433 2d35 3534 3033 2d36 3235 3637 2d30  :6-16777343-55403-62567-0
    00000228:  3a18 736c 6176 6528 3129 4031 3237 2e30 2e30 2e31 3a35 3534  ::.slave(1)@127.0.0.1:554
    00000240:  3033 4000
    
#Record-IO 序列化/反序列化例子
用 Python 如何发送和接送 record-io 格式化的信息
代码片段来自于  src/examples/python/test_containerizer.py

    #Read a data chunk prefixed by its total size from stdin.
    def receive():
        # Read size (uint32 => 4 bytes).
        size = struct.unpack('I', sys.stdin.read(4))
        if size[0] <= 0:
            print >> sys.stderr, "Expected protobuf size over stdin. " \
                            "Received 0 bytes."
            return ""
        # Read payload.
        data = sys.stdin.read(size[0])
        if len(data) != size[0]:
            print >> sys.stderr, "Expected %d bytes protobuf over stdin. " \
                            "Received %d bytes." % (size[0], len(data))
            return ""
    return data
    # Write a protobuf message prefixed by its total size (aka recordio)
    # to stdout.
    def send(data):
        # Write size (uint32 => 4 bytes).
        sys.stdout.write(struct.pack('I', len(data)))
        # Write payload.
        sys.stdout.write(data)
        


  [1]: http://mesos.apache.org/documentation/latest/external-containerizer/#sandbox
  [2]: http://mesos.apache.org/documentation/latest/external-containerizer/#record-io-deserializing-example
  [3]: https://github.com/apache/mesos/blob/master/docs/images/ec_launch_seqdiag.png?raw=true
  [4]: https://raw.githubusercontent.com/apache/mesos/master/docs/images/ec_lifecycle_seqdiag.png
  [5]: https://raw.githubusercontent.com/apache/mesos/master/docs/images/ec_kill_seqdiag.png
  [6]: https://github.com/apache/mesos/blob/master/docs/images/ec_recover_seqdiag.png?raw=true
  [7]: https://github.com/apache/mesos/blob/master/docs/images/ec_orphan_seqdiag.png?raw=true
  [8]: http://mesos.apache.org/documentation/latest/external-containerizer/#slave-recovery-overview
  [9]: http://mesos.apache.org/documentation/latest/external-containerizer/#enhanced-verbosity-logging