
#在虚拟机中开发Marathon

 

基于开发的目的， 不需要编译和配置一个本地的Mesos环境，在虚拟机开发将使Marathon运行在本地。


1.确定本地的Marathon已经编译和安装。 在  marathon  目录 ：
   
    sbt assembly

2.希望看到本地Marathon使用Javascript进行自定义么？ 如果是这样您需要编译assets。这里是  assets的工作指南  [Compiling Assets](https://github.com/mesosphere/marathon-ui#compiling-assets)。

3.克隆  [playa-mesos repository](https://github.com/mesosphere/playa-mesos)项目。需要注意的是playa-mesos项目附带的Mesos  Marathon和ZooKeeper需要重新配置。


4.同步本地的文件夹到 Vagrant  镜像。打开  `playa-mesos/Vagrantfile`  并编辑      
`override.vm.synced_folder``config.vm.synced_folder '</path/to/marathon/parent>', '/vagrant'`
      

5.SSH 到你的  Vagrant  镜像

    $ vagrant up # if not already running otherwise `vagrant reload`
    $ vagrant ssh


6.检查你的文件夹同步是否正确


    $ cd /vagrant/
    $ ls
    marathon playa-mesos ...

7.在Vagrant镜像中停止 Marathon 

    $ sudo stop marathon

8.通过增加  `marathon-start`  启动你自己版本的  Marathon

 `$ nano ~/.bash_aliases`

9.在这个文件的顶部添加以下的内容并保存：

    # setup marathon easy run
    alias 'start-marathon'='./bin/start --master zk://localhost:2181/mesos --zk zk://localhost:2181/marathon --assets_path src/main/resources/assets'

10.刷新终端并在 marathon  文件夹下运行   `start-marathon`  命令


    $ . ~/.bashrc
    $ cd /vagrant/marathon/
    $ start-marathon


在您的浏览器加载 Marathon UI： http://10.141.141.10:8080， 当完成后，通过  `vagrant halt` 来关闭Vagrant实例。
