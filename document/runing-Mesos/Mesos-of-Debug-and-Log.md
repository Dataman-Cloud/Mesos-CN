#日志与调试

通过使用 Google Logging library，Mesos 默认将 log 写到***MESOS\_HOME/logs*** 目录（***MESOS\_HOME*** 是本地已安装 Mesos 的路径）。log 目录可以通过 ***log\_dir*** 变量进行配置。

运行在 Mesos 上的 Frameworks 将它们的输出内容储存到每个机器上的一个叫" work "的目录，默认情况下，就是 ***Mesos\_HOME/work*** 目录。以这个目录为基础，frameworks 的输出被以 ***slave-X/fw-Y/Z*** 格式放置到 ***stdout*** 和 ***stderr*** 目录下，其中 ***X***是 slave ID ，Y 是 framework ID。每次试图运行 framwork 的 executor 时，则会创建一个子目录 Z。 这些目录也可以通过 slave daemon 的 Web UI 进行访问。
>内容翻译自:[http://mesos.apache.org/documentation/latest/logging-and-debugging/](http://mesos.apache.org/documentation/latest/logging-and-debugging/)
