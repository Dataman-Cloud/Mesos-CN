## Docker Containerizer

Mesos 0.20.0 开始支持通过 Docker 镜像容器来启动任务，同时也支持部分的 Docker 参数。当然我们计划在未来支持更多的参数。

用户可以将 Docker 镜像作为一个任务启动，也可以作为一个 Executor 启动。

以下部分将描述 API 的变化以及支持 Docker 的新功能，还有如何设置 Docker。

###设置
为了运行支持 Docker 容器的 slave，在启动slave的时候，你必须将 " docker" 作为容器化的选项之一 。

例子: mesos-slave –containerizers=docker,mesos


每个支持 Docker 容器化的 slave 都应该已经安装了 Docker CLI 客户端 ( 版本号 > = 1.0.0)。


###如何使用 Docker 容器化 

TaskInfo 在 0.20.0 版本前仅适用于支持两种方式。 一种是通过 CommandInfo 启动运行 bash 命令任务，另一种是通过 ExecutorInfo 来启动自定义的执行任务。

随着 0.20.0 后，我们增加了一个 TaskInfo 和 ExecutorInfo 的 ContainerInfo 字段，允许 Containerizer 作为 Docker 配置来运行 task 或 executor。

要运行一个 Docker 镜像作为 task ( 任务 )，在 TaskInfo 中必须设置 command 和 container 字段作为 Docker Containerizer ，并将其作为启动 docker 镜像的辅助命令。 ContainerInfo 应该含有 Docker 类型和一个 docker image 所需的 DockerInfo 。

要运行一个 Docker 镜像作为 executor （ 执行器 ），在 TaskInfo 中必须设置包含 ContainerInfo  类型的 docker 。并且， CommandInfo  将被用于启动 executor 。

###Docker Containerizer 的作用

Docker Containerizer 转化为 Task / Executor 启动，Docker CLI 命令用于销毁。

目前，Docker Containerizer 作为任务启动时，需要执行以下操作：

1. 获取所有指定 CommandInfo 进入到沙盒中的文件
2. 从远程仓库拉取 docker 镜像
3. 运行 docker 镜像作为 Docker executor ，映射沙盒目录到 Docker container 中，并且设置目录映射到 MESOS_SANDBOX 环境变量。 executor 也将容器日志以流的方式输入/输出到沙箱中的文件。
4. 当 container 退出，或者 containerizer 销毁，停止并移除 docker 容器。

 Docker Containerizer 优先启动所有以 " mesos- " 为前缀的 slave ID （ 如，mesos-slave1-abcdefghji ），并且假设所有以 " mesos- "为前缀的容器由 slave 管理，可以自由的停止和销毁容器。

当启动 docker 镜像为 Executor。唯一的区别是，它跳过启动执行命令，但是将获取 docker 容器执行器的 pid。

注意，我们目前默认主机网络在运行一个 docker 容器，以支持更容易运行的 docker 镜像作为 Executor 。

###私有 Docker 仓库
若要从私有仓库运行一个镜像，需 uri 指向包含登录信息的  .dockercfg 文件  。.dockercfg 文件将被 pull 到沙盒中。  Docker Containerizer 设置 HOME 环境变量指向沙盒。所以  docker cli 会自动配置文件。

###CommandInfo to run Docker images
一个 docker 镜像目前支持入口点为 and / or 作为一个默认命令。

若要用默认命令运行一个 docker 镜像 （如，docker run image ）， CommandInfo 的 value  不能被设置。如果 value 被设置，那么它将覆盖默认的命令。

若要用指定的入口运行 docker 镜像，CommandInfo 的 shell 选项必须设置为 false 。如果设置为 true , Docker Containerizer 运行的用户命令  /bin/sh -c 也将成为镜像的入口。

###Recover Docker containers on slave recovery

无论 slave 是否在 Docker 容器中运行。当 slave 重启时，Docker containerizer 都支持恢复 Docker 容器。
