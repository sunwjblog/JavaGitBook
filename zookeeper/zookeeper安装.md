# zookeeper安装

### 下载zookeeper压缩包

[zookeeper压缩包下载](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz)

### 解压压缩包

### 修改配置文件

```
> cd apache-zookeeper-3.7.0-bin/conf //切换到配置目录下
> mv zoo_sample.cfg zoo.cfg //更改默认配置文件名称
> vi zoo.cfg //编辑配置文件，自定义dataDir
```

### 启动zookeeper

单独进入bin路径下直接启动可能会报错，如：

```
> ./zkServer.sh start //启动
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /Users/admin/zookeeper/apache-zookeeper-3.5.6/bin/../conf/zoo.cfg
Starting zookeeper ... FAILED TO START
```

需要这样启动

```
bin/zkServer.sh start conf/zoo.cfg

/Users/sunwj/Documents/sunwj/soft/apache-zookeeper-3.7.0-bin/bin/zkServer.sh start /Users/sunwj/Documents/sunwj/soft/apache-zookeeper-3.7.0-bin/conf/zoo.conf

/Users/sunwj/Documents/sunwj/soft/apache-zookeeper-3.7.0-bin/bin/zkServer.sh stop /Users/sunwj/Documents/sunwj/soft/apache-zookeeper-3.7.0-bin/conf/zoo.conf
```

