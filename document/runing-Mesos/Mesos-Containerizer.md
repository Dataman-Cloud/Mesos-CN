## Mesos Containerizer
&nbsp;&nbsp;&nbsp;&nbsp;Mesos Containerizer 提供轻量级的容器化，其使用了 Linux 特有的功能，如 cgroup 和 namespace 。并且是可以组合的，因此可以选择性的使用不同的 isolators 。

&nbsp;&nbsp;&nbsp;&nbsp;Mesos Containerizer 还提供了对 POSIX 系统的基本支持 （ 如 OS X ），但没有实际的隔离效果，只是提供了资源的使用情况报告。

### Shared Filesystem （ 共享文件系统 ）
&nbsp;&nbsp;&nbsp;&nbsp;Shared Filesystem 可以被任意使用在 Linux hosts 上， 使每个修改共享文件的 container 都可以看到。 

&nbsp;&nbsp;&nbsp;&nbsp; 通过修改指定的包含在 ExecutorInfo 中的 ContainerInfo ，无论是通过 framework 或者使用 ***–default_container_info*** 的 slave 标记。

&nbsp;&nbsp;&nbsp;&nbsp;我们将 ContainerInfo 指定的 Volumes 以读写或只读的方式，映射部分 shared filesystem ( host\_path ) 到容器的 filesystem ( container\_path ) 中。host\_path 的路径可以是绝对的，在这种情况下，它可以使文件系统的子目录映射到 host\_path 下。如果 host\_path 是相对路径，那么它可以被看做为一个 executor 的工作目录。将创建的目录和权限从相应的目录( 必须存在 )复制到共享文件系统中。

&nbsp;&nbsp;&nbsp;&nbsp;对于 isolator（ 隔离器 ），最主要的目的是有效的用共享文件系统中的一部分为每个 container 提供专有的空间。例如，一个私有的 "/temp " 目录可以通过 host\_path = " tmp " 和 container\_path = " /temp " 实现。这将在 excutor 的工作目录中创建一个 "tmp" 目录( 1777 权限 )，同时在 container 中将其挂载为 " /tmp "。在 container 的运行过程中是透明的。Containers 将不会看到 host 的 / tmp 或者任何别的容器的 / tmp 。

###Pid Namespace

&nbsp;&nbsp;&nbsp;&nbsp;Pid Namespace 隔离器在一个单独的 pid namespace 中，可用来隔离每个 container。它有两个主要的优点：

&nbsp;&nbsp;&nbsp;&nbsp;1. 可见性：进程在容器 ( executor 和 descendants ) 中运行时，是无法看到或者给 namespace 外部的进程发信号的。

&nbsp;&nbsp;&nbsp;&nbsp;2. Clean termination（ 可以理解为清理后事 ）：当终止一个主要进程的 pid namespace 时，内核同时也会终止所有其他的 namespace 。


&nbsp;&nbsp;&nbsp;&nbsp;利用第 2 个优点，它会避免目前内核已知的一个会引起 OOM 的 BUG 。


&nbsp;&nbsp;&nbsp;&nbsp;/ proc 将被挂载到 container ，所以一些例如 " ps " 之类的工具仍可以正常使用。

###Posix 磁盘隔离

&nbsp;&nbsp;&nbsp;&nbsp;Posix 磁盘隔离提供基本的磁盘隔离。它可以提供每个沙盒的硬盘使用情况和可分配的磁盘额度。Posix 磁盘隔离机制可以在 Linux 和 OS X 中使用。

&nbsp;&nbsp;&nbsp;&nbsp;使用 Posix 磁盘隔离需要在 slave 启动的时候在 ***posix/disk*** 后添加 ***--isolation*** 标志。

&nbsp;&nbsp;&nbsp;&nbsp;默认情况下是禁止使用磁盘分配的。如果要使用的话，需要当 slave 启动时指定 ***--enforce_container_disk_quota*** 。

&nbsp;&nbsp;&nbsp;&nbsp;Posix Disk isolator (Posix 磁盘隔离器) 通过每个沙盒的 ***du*** 命令来定期报告磁盘的使用情况。磁盘的使用情况可以从资源检索统计端点 ( ***/monitor/statistics.json*** ) 统计。

&nbsp;&nbsp;&nbsp;&nbsp;两个 ***du*** 命令之间的间隔可以通过 slave 标记 ***--container_disk_watch_interval*** 来指定。例如，***--container_disk_watch_interval=1mins*** 设置间隔为 1 MIN 。默认情况下间隔为 15 S。