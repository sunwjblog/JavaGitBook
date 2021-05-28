# Redis安装mac

## 利用Homebrew安装Redis

### 安装命令

```
brew install redis
```

### 安装目录

安装成功之后，进入安装目录进行启动。

```shell
sunwj@sunwjdeMacBook-Pro  ~/Documents/GitHub/JavaGitBook   main ±  cd /usr/local/Cellar/redis/6.0.6/bin
sunwj@sunwjdeMacBook-Pro  /usr/local/Cellar/redis/6.0.6/bin  ll
total 7800
lrwxr-xr-x  1 sunwj  admin       12  5 28 11:16 redis-sentinel -> redis-server
-r-xr-xr-x  1 sunwj  admin  1129544  5 28 11:16 redis-check-aof
-r-xr-xr-x  1 sunwj  admin  1129544  5 28 11:16 redis-check-rdb
-r-xr-xr-x  1 sunwj  admin   265720  5 28 11:16 redis-cli
-r-xr-xr-x  1 sunwj  admin   333696  5 28 11:16 redis-benchmark
-r-xr-xr-x  1 sunwj  admin  1129544  5 28 11:16 redis-server
```

#### 遇到的问题

```
 ✘ sunwj@sunwjdeMacBook-Pro  /usr/local/Cellar/redis/6.0.6/bin  redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> shutdown
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected>
```

解决方案：

需要配置redis.conf配置问题

```
vi /usr/local/etc/redis.conf
```

将daemonize no 改为 daemonize yes

然后执行命令

```shell
sunwj@sunwjdeMacBook-Pro  /usr/local/Cellar/redis/6.0.6/bin  redis-server /usr/local/etc/redis.conf
sunwj@sunwjdeMacBook-Pro  /usr/local/Cellar/redis/6.0.6/bin  redis-cli
```

即可进入redis实例中。

#### 设置密码

```
sunwj@sunwjdeMacBook-Pro  /usr/local/Cellar/redis/6.0.6/bin  redis-cli
127.0.0.1:6379> auth 19920610
(error) ERR AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
127.0.0.1:6379> AUTH sunwjredis
(error) ERR AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
127.0.0.1:6379> config set requirepass "admin"
OK
127.0.0.1:6379> auth admin
OK
127.0.0.1:6379>
```

#### 连接redis服务

```
redis-cli -h 127.0.0.1 -p 1234
```

#### 退出本次会话

```
127.0.0.1:6379>quit
```

#### 关闭服务

```
127.0.0.1:6379>shutdown save|nosave
```

