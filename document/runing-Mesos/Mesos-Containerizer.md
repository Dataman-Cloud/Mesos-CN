## Mesos Containerizer
Mesos Containerizer 提供轻量级的容器化和资源隔离功能，主要使用了 Linux 内建的 cgroup 和 namespace 等机制 。这些机制可以组合使用，来满足不同的的隔离需求。

Mesos Containerizer 还提供了对 POSIX 系统的基本支持（ 如 OS X ），但不支持资源隔离功能，只是提供资源的使用情况报告。

### Shared Filesystem （ 共享文件系统 ）
共享文件系统隔离器（Shared Filesystem isolator）可以被选装在 Linux 宿主机上，这样 container 和宿主机之间可以共享文件系统。 

有关共享文件系统的配置信息位于 ExecutorInfo 中的 ContainerInfo 段落，无论是通过 framework 或者启动 slave 的时候使用 ***–default_container_info*** 标志位。

ContainerInfo 中的 Volumes 段落定义了将宿主机部分共享文件路径 ( host\_path ) 映射到容器内的文件路径 ( container\_path ) 上，权限可以是“读写”或“只读”。host\_path 可以是绝对路径，宿主机该路径下的所有子目录都会映射到所有容器内的 container\_path 下。如果 host\_path 是个相对路径，表示相对于宿主机内的 executor 工作目录的位置。运行时会在宿主机的这个位置新建一个目录，其文件访问权限会和容器内所映射的目录的一致（容器内那个目录必须已经存在）。

引入共享文件系统隔离器（Shared Filesystem isolator）的目的，是让每个容器能都拥有专有的目录空间。容器私有的 "/temp" 目录可以这么来设置参数 —— host\_path = "tmp" 和 container\_path = "/temp "。结果在宿主机 excutor 的工作目录下创建一个 "tmp" 目录(1777 权限)，同时在容器内映射到根目录下 "/tmp" 之下。容器内的进程将看不见宿主机的根目录 /tmp 或者其他容器内的 /tmp 。

###Pid Namespace

Pid 命名空间隔离器（Pid Namespace isolator ）将每个容器分割在单独的命名空间。它有两个主要的优点：

1. 可见性：在容器中运行的进程 ( executor 和 descendants ) ，无法看到或者向命名空间之外的进程发信号。

2. Clean termination：当在命名空间内终止一个 leading process，内核同时也会终止该命名空间内其他的进程 。

The Launcher will use (2) during destruction of a container in preference to the freezer cgroup, avoiding known kernel issues related to freezing cgroups under OOM conditions.

目录 /proc 将被挂载到容器之中，所以 "ps" 之类的工具仍可以使用。

###Posix 磁盘隔离器

Posix 磁盘隔离器（Posix Disk isolator）提供基本的磁盘隔离功能。它可以提供每个沙盒的硬盘使用信息，必要时也可以用来限制磁盘使用额度。Posix 磁盘隔离机制支持 Linux 和 OS X。

启用 Posix 磁盘隔离，需要在启动 slave 时在 ***posix/disk*** 标志位后添加 ***--isolation***。

默认情况下没有启用磁盘额度限制功能。如果要启用的话，需要在启动 slave 时添加 ***--enforce_container_disk_quota*** 参数。

Posix 磁盘隔离器通过定期在每个沙盒内执行 ***du*** 命令来报告磁盘的使用情况。磁盘的使用情况可以资源统计端点( ***/monitor/statistics.json*** ) 查询。

两次执行 ***du*** 命令的间隔可以通过 slave 标记 ***--container_disk_watch_interval*** 来指定。例如 ***--container_disk_watch_interval=1mins*** 设置间隔为 1 MIN ，默认情况下间隔为 15 秒。
