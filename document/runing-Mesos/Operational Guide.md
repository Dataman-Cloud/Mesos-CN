#Operational Guide

#操作指南

Currently the master leverages a paxos-based replicated log as its storage backend (--registry=replicated_log is the only storage backend supported). Each master participates in the ensemble as a log replica. The --quorum flag determines a majority of the masters.

目前， master利用paxos-based复制log作为存储后端（`--registry=replicated_log`仅仅支持后端存储）。每个master作为日志的副本。`--quorum`决定了大多数的master。


The following table shows the tolerance to master failures, for each quorum size:

下表显示了tolerance对master failures， 每一个quorum的大小。


Masters      Quorum Size	 Failure Tolerance
1	              1          	0
3                 2	            1
5	              3         	2
…	…	…
2N - 1	N	N - 1


When configuring the quorum, it is essential to ensure that there are only so many masters running as specified in the table above. If additional masters are running, this violates the quorum and the log may be corrupted! As a result, it is recommended to gate the running of the master process with something that enforces a static whitelist of the master hosts. See [MESOS-1546](https://issues.apache.org/jira/browse/MESOS-1546) for adding a safety whitelist within Mesos itself.

在配置quorum时， 确保只有这么多的master指定运行在上表中是至关重要的。 如果额外的master运行，这违反了quorum和log可能会损坏！作为一个结果，建议运行master进程执行一个静态主机的白名单。浏览[MESOS-1546](https://issues.apache.org/jira/browse/MESOS-1546)增加一个安全的白名单在Mesos本身。

For online reconfiguration of the log, see: [MESOS-683](https://issues.apache.org/jira/browse/MESOS-683).

对于在线重新配置的日志， 浏览[MESOS-683](https://issues.apache.org/jira/browse/MESOS-683).

Increasing the quorum size

增加quorum的大小

As the size of a cluster grows, it may be desired to increase the quorum size for additional fault tolerance.

随着集群的大小，这可能需要增加quorum的大小

The following steps indicate how to increment the quorum size, using 3 → 5 masters as an example (quorum size 2 → 3):

以下步骤显示如何增加quorum的大小，将使用masters从3个增加到5个作为一个例子（quorum的大小从2个增加到3个）



1. Initially, 3 masters are running with `--quorum=2`


2. Restart the original 3 masters with `--quorum=3`


3. Start 2 additional masters with `--quorum=3`
To increase the quorum by N, repeat this process to increment the quorum size N times.



----------


1. 最初，3个master正在运行 `--quorum=2`


2. 重新启动原始的3个master `--quorum=3`


3. 开启2个额外的master `--quorum=3`
增加quorum到N， 重复这过程N次来达到quorum大小增加到N。


Decreasing the quorum size

降低quorum的大小


The following steps indicate how to decrement the quorum size, using 5 → 3 masters as an example (quorum size 3 → 2):

以下步骤显示如何减少quorum的大小，将使用masters从5个降低到3个作为一个例子（quorum的大小从3个降低到2个）




1. Initially, 5 masters are running with `--quorum=3`


1. Remove 2 masters from the cluster, ensure they will not be restarted (See NOTE section above). Now 3 masters are running with `--quorum=3`


1. Restart the 3 masters with `--quorum=2`

----------

1. 首先，5个master正在运行 `--quorum=3`
2. 从集群中移除2个master， 确保他们将不重启（参见上边的备注部分）。现在3个master正在运行 `--quorum=3`
3. 重启这3个master `--quorum=2`

To decrease the quorum by N, repeat this process to decrement the quorum size N times.

减少quorum到N， 重复这过程N次来使得quorum大小减少到N个。

Replacing a master

替换一个master

Please see the NOTE section above. So long as the failed master is guaranteed to not re-join the ensemble, it is safe to start a new master with an empty log and allow it to catch up。

请参阅上面的备注部分。只要保证failed master不重新加入，开启一个新的master与一个空的日志是安全的， 



