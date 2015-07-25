#Framework-Rate-Limiting

Framework rate limiting 是 Mesos 0.20.0 引入的新的特色。

####什么是 Framework Rate Limiting

在多框架环境中，这个功能的主要目的是通过 master 限制其他框架的消息 ( 如 development, batch ) ， 来保护 high-SLA 框架 ( 如 production, service )的吞吐量。

在一个framework中限制message流量时, Mesos 集群管理员通过设置 ***qps*** 值（queries per seconds）来确定每个 framework 的 principal。 当每秒访问次数超过了 ***qps*** 的值后，它将不处理超过的这些 message 。 这些未处理的　message 将保存在 master 的内存中。

####速率限制的配置

下面是一个简单的配置文件 ( JSON 格式 )，可以指定 ***--rate-limits*** master 标志。
```
{
  "limits": [
    {
      "principal": "foo",
      "qps": 55.5
      "capacity": 100000
    },
    {
      "principal": "bar",
      "qps": 300
    },
    {
      "principal": "baz",
    }
  ],
  "aggregate_default_qps": 333,
  "aggregate_default_capacity": 1000000
}
```

这个例子中， framework ***foo*** 在配置中限制了 ***qps*** 和 ***capacity*** ，framework ***bar*** 没有限制　capacity　，framework ***baz*** 则没有任何限制。如果有第四个 framework ***qux*** 或者一个没有 principal 的 framewrok 连接到 master，则他们通过 ***aggregate_default_qps*** 和 ***aggregate_default_capacity*** 规则来限流。

###配置说明

下面是 JSON 配置的一些字段

* ***principal*** ( 必需 ) 被限流或者指定为不限制的唯一标识。
* 它需要和 framework 的 FrameworkInfo.principal 相匹配。
* 可以多个 framework 使用相同 principal 。这种情况下,所有使用相同 principal 的framework 合流并公用指定的 QPS 。
* ***qps*** ( 可选 ) 每秒查询数。即 rate。
* 一但设定， master 保证该　principal　中处理信息的速度不会超过该值。然而　master 可以比这个速度慢，尤其是当 rate 过高。
* 为了明确的指定一个　framework 不需要限制　rate ,需要添加一个 ***limits*** 标志。
* ***capacity*** （ 可选）master 允许 principal 中 framework 未处理 message 的数量。如果没有指定，则没有数量限制。注意，如果不限制或者限制的数字太大，可能会引起　master 的 OOM 。
* 注意：如果没有指定 ***qps*** , ***capacity*** 则会被忽略。
* 使用 ***aggregate_default_qps*** 和 ***aggregate_default_capacity*** 来保护 master。所有未用 ***limits***限流的 framework 都将使用默认的 rate 和　capacity 。
* rate和capacity都会将这些值聚合起来，也就是说，他们合并后的流量会一起被限制。
* 和上面一样，若 ***aggregate_default_qps*** 没有指定，则 ***aggregate_default_capacity*** 将被忽略。
* 如果这些字段没有被指定，则未指定的 framework 不限流。我们建议使用这些选项，特别是当 master 不需要身份验证的时候，可以防止意想不到的 framewrok 压垮　master。

###使用 Framework Rate Limiting
####Monitoring Framework Traffic
当一个framework注册到master当中时,master会在它的metrics当中显示出所有从那个framework接受和处理过的消息指标 endpoint: http://<master>/metrics/snapshot. 例如， framework ***foo*** 有两个消息计数器 ***frameworks/foo/messages_received*** 和 ***frameworks/foo/messages_processed*** 。 没有限制 framework 的 rate ，则这两个数量应该相差不多或者相差很少（ 因为消息都会被尽快处理掉 ），但是当一个 framework 被限流，则它们之间的差值就是未处理的消息。

通过持续不断的监视计数器，你可以得到这个 framework 消息到达的速率和消息队列的增长率 （ 如果它被限流的话 ）。 这描述了在 framework 的网络流量方面的特征。

####Rate Limits 配置

使用 framework 限流是为了防止 low-SLA framework 使用太多的资源并且尽可能精确的保证不会重复他们的流量和行为，你可以通过使用大点的 ***qps*** 值来限流。 实际上它们已经对来自high-SLA 的框架中高优先级的消息进行了有效的限流，因为他们可以更快的处理更高的优先级 。

要计算 master 有多少 ***capacity*** 可以被处理，你需要知道 master 进程的内存限制，内存的数量通常和没有速率限制的服务的工作量 (例如, 使用 ps -o rss $MASTER_PID) 以及framework中消息的平均大小类似 (队列消息以带有一些附加字段的序列化协议缓冲Buffers的形式进行存储) 你应该在配置中总结所有的capacity值 。无论如何，这种计算都不是精确的。起初你应该设置较小的值，以留有足够的余地。

####处理 “Capacity Exceeded” 错误

当一个 framework 超过 ***capacity*** 容量，将返回一个 ***FrameworkErrorMessage***。它不会杀死任何　task 和 scheduler 本身。 框架开发者可以选择重启或者故障调度机制来弥补被丢弃的消息。（ 除非你的 framework  不关心所有发送到 master 的消息都要被处理 ）。

在 0.20.0 版本后，我们打算迭代一个 master 早期预警的功能。 scheduler 可以限制自己的流量己或者忽略警告，如果它被设计为临时的突发情况 。

在使用提前预警功能时，我们不建议在生产环境中使用 rate limiting 。除非你明确错误的后果。 当然，在限制其他frameworks的流量前提下使用它去保护生产环境的frameworks是没问题的，并且如果它没有被显示的启动那么在master上是不会受到任何影响的。

>本文翻译自http://mesos.apache.org/documentation/latest/framework-rate-limiting/


