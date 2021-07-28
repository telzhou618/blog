# 3.Redis从入门到放弃

## Redis 是什么

Redis 是一个高性能的基于内存实现的K-V存储数据库 。

## Redis 的优点

- 性能极高，Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 支持持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- 多种数据类型，同时还提供list，set，zset，hash等数据结构的存储。
- 支持主从复制，自动同步，可实现读写分离。
- 丰富的特性，支持pub/sub，key过期策略，事务，支持多个DB等。

## Redis 的缺点

- 基于内存存储，单机容量有限。
- 重启会加载磁盘缓存到内存，这期间不能提供服务。
- 主从复制采用全量复制，这个过程会占用更多内存和网络资源。
- 不支持自动容错，主从任意一个宕机会造成前端读写失败。

## 场景引用场景

- 缓存数据，常见应用。

- 计数器，单线程可避免并发，保证不出错，性能毫秒级。
- 队列，可作为简单消息队列使用。
- 分布式锁与单线程机制
- 位操作，如统计在线列表
- 最新列表，获取最新10条新闻数据
- 排行榜，消费排行榜

## Redis 基础

### 安装

```shell
# 把下载好的redis-5.0.3.tar.gz放在/usr/local文件夹下，并解压
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar xzf redis-5.0.3.tar.gz
cd redis-5.0.3

# 进入到解压好的redis-5.0.3目录下，进行编译与安装
make

# 修改配置
daemonize yes  #后台启动
protected-mode no  #关闭保护模式，开启的话，只有本机才可以访问redis
# 需要注释掉bind
#bind 127.0.0.1

# 启动服务
src/redis-server redis.conf

# 验证启动是否成功 
ps -ef | grep redis 

# 进入redis客户端 
src/redis-cli 

# 退出客户端
quit

# 退出redis服务： 
（1）pkill redis-server 
（2）kill 进程号                       
（3）src/redis-cli shutdown 
```

### 持久化

#### 持久化 - rdb 模式

开启rdb模式

```she
# save 60 1000 //关闭RDB只需要将所有的save保存策略注释掉即可
```

还可以手动执行命令生成RDB快照，进入redis客户端执行命令**save**或**bgsave**可以生成dump.rdb文件， 每次命令执行都会将所有redis内存快照到一个新的rdb文件里，并覆盖原有rdb快照文件。

#### 持久化 - aof模式

开启aof模式

```sh
 appendonly yes
```

aof策略配置

```sh
1.appendfsync always：每次有新命令追加到 AOF 文件时就执行一次 fsync ，非常慢，也非常安全。
2 appendfsync everysec：每秒 fsync 一次，足够快，并且在故障时只会丢失 1 秒钟的数据。
3 appendfsync no：从不 fsync ，将数据交给操作系统来处理。更快，也更不安全的选择。
```

## Redis 实例

## Redis主从模式

<img src="/_static/images/image-20210728155319971.png" alt="image-20210728155319971" style="zoom:50%;" />

### 主从模式配置

```sh
1、复制一份redis.conf文件

2、将相关配置修改为如下值：
port 6380
pidfile /var/run/redis_6380.pid  # 把pid进程号写入pidfile配置的文件
logfile "6380.log"
dir /usr/local/redis-5.0.3/data/6380  # 指定数据存放目录
# 需要注释掉bind
# bind 127.0.0.1（bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）

3、配置主从复制
replicaof 192.168.0.60 6379   # 从本机6379的redis实例复制数据，Redis 5.0之前使用slaveof
replica-read-only yes  # 配置从节点只读

4、启动从节点
redis-server redis.conf

5、连接从节点
redis-cli -p 6380

6、测试在6379实例上写数据，6380实例是否能及时同步新修改数据

7、可以自己再配置一个6381的从节点
```

### 主从工作原理

如果你为master配置了一个slave，不管这个slave是否是第一次连接上Master，它都会发送一个**PSYNC**命令给master请求复制数据。

master收到PSYNC命令后，会在后台进行数据持久化通过bgsave生成最新的rdb快照文件，持久化期间，master会继续接收客户端的请求，它会把这些可能修改数据集的请求缓存在内存中。当持久化进行完毕以后，master会把这份rdb文件数据集发送给slave，slave会把接收到的数据进行持久化生成rdb，然后再加载到内存中。然后，master再将之前缓存在内存中的命令发送给slave。

当master与slave之间的连接由于某些原因而断开时，slave能够自动重连Master，如果master收到了多个slave并发连接请求，它只会进行一次持久化，而不是一个连接一次，然后再把这一份持久化的数据发送给多个并发连接的slave。

## Redis 哨兵模式

## Redis 集群模式

## Redis 核心原理

## Redis 常见问题及解决

## Redis 参考

