## Docker Containerizer（用 Docker 容器部署应用）

Mesos 0.20.0 开始支持通过 Docker 镜像来启动任务，同时也支持部分的 Docker 参数。当然我们计划在未来支持更多的参数。

用户可以将 Docker 镜像作为一个任务启动，也可以作为一个 Executor 启动。

以下部分将描述 API 的变化以及支持 Docker 的新功能，还有如何设置 Docker。

###设置
为了运行支持 Docker 容器的 slave，在启动slave的时候，你必须将 " docker" 作为 Containerizer （ 容器化 ）选项之一 。

例子: mesos-slave –containerizers=docker,mesos


每个支持 Docker Containerizer 的 slave 都应该已经安装了 Docker CLI 客户端 ( 版本号 > = 1.0.0)。


###如何使用 Docker 容器化 

在 0.20.0 版本之前，TaskInfo 仅适用于两种情况：一种是通过 CommandInfo 启动运行 bash 命令任务，另一种是通过 ExecutorInfo 来调用自定义的执行器（ Executor ）来启动任务。（TODO）

随着 0.20.0 后，我们在 TaskInfo 和 ExecutorInfo 里各增加了 1 个 ContainerInfo 字段，可以设置为容器（ 例如 “Docker” ）来运行 task 或 executor。

要通过 Docker 镜像来运行任务 ( task )，在 TaskInfo 中必须设置 command 和 container field 字段， Docker Containerizer 会将它们作为启动 docker 镜像的辅助命令。 ContainerInfo 中的容器类型要设置为 “Docker”，而 DockerInfo 则指定要启动的 docker 镜像信息。

要运行 Docker 镜像作为 executor （执行器），在 TaskInfo 中必须设置包含 ContainerInfo  类型的 docker 。并且， CommandInfo  将被用于启动 executor 。

###Docker Containerizer 是如何工作的

Docker Containerizer 将对 Task / Executor 的启动和销毁操作映射成 Docker CLI 的命令。

目前，Docker Containerizer 作为任务启动时，需要执行以下操作：

1. 将所有在 CommandInfo 中指定的文件放入沙盒中
2. 从远程仓库拉取 docker 镜像
3. 使用  Docker executor 来运行 docker 镜像，同时将沙盒目录映射到容器中环境变量 MESOS_SANDBOX 所指定的目录。 executor 也将容器的日志流重新定向到沙箱中的 stdout/stderr 文件。
4. 当退出容器或者销毁 containerizer 时，停止并移除 docker 容器实例。

 Docker Containerizer 启动的容器的 ID 由 前缀" mesos- " 加上 slave ID（ 如，mesos-slave1-abcdefghji ）组成，并且假设所有以 " mesos- "为前缀的容器都由 slave 管理，slave 可以任意停止和销毁容器。

当启动 docker 镜像为 Executor。唯一的区别是，它跳过启动执行命令，但是将获取 docker 容器执行器的 pid。（todo）

注意，为了更方便地让 docker 镜像充当 Executor，我们目前默认以 host 的网络模式来在运行 docker 容器 。

containerizer 也支持强制从镜像仓库更新 docker 镜像。如果我们关闭了这个功能，只有在 slave 上没有待运行的 docker 镜像或者镜像已经被更新的时候， slave 才会主动去拉取。

###私有 Docker 仓库
若要从私有仓库运行一个镜像，需要指向包含登录信息的 .dockercfg 文件的 uri，这样 .dockercfg 文件会被下载到沙盒中。因为 Docker Containerizer 设置了指向沙盒的 HOME 环境变量，所以 Docker CLI 会自动识别这个配置文件。

###CommandInfo to run Docker images
docker 镜像目前支持 dockerfile 中定义的 ENTRY 和 CMD 命令。

如果要在运行 docker 镜像时启用 CMD 参数（如，docker run image ），则不能设置 CommandInfo，否则它将覆盖 CMD 定义的命令。

若要用指定的 ENTRY 运行 docker 镜像，CommandInfo 的 shell 选项必须设置为 false。如果设置为 true , Docker Containerizer 运行的用户命令  /bin/sh -c 也将成为镜像的入口。

###Slave 节点重启的时候恢复容器

无论 slave 本身是否在 Docker 容器中运行。当 slave 重启时，Docker containerizer 都支持恢复之前处于运行状态的容器。

如果 Docker containerizer 自身就运行在容器中，请把 flag —— docker_mesos_image 被设置为 true。
