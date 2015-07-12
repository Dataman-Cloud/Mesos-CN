#日志与调试

Mesos 使用 Google Logging library 并且默认将 log 写到***MESOS\_HOME/logs*** 目录，***MESOS\_HOME*** 是本地已安装 Mesos 的路径。log 目录可以通过 ***log\_dir*** 进行配置。

运行在 Mesos 上的 Frameworks 将它们的输出储存到每个机器上的一个叫" work "的目录。默认情况下，就是 ***Mesos\_HOME/work*** 目录。在这个目录中，frameworks 的输出被以 ***slave-X/fw-Y/Z*** 格式放置到 ***stdout*** 和 ***stderr*** 目录中，其中 ***X***是 slave ID ，Y 是 framework ID,多个子目录 Z 则是每个试图运行 framwork 的 executor 。 这些目录也可以通过 slave daemon 的 Web UI 进行访问。
>内容翻译自:[http://mesos.apache.org/documentation/latest/logging-and-debugging/](http://mesos.apache.org/documentation/latest/logging-and-debugging/)
