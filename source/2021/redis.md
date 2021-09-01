# Redis 从入门到放弃

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

## 应用场景

- 缓存数据，常见应用。

- 计数器，单线程可避免并发，保证不出错，性能毫秒级。
- 队列，可作为简单消息队列使用。
- 分布式锁与单线程机制
- 位操作，如统计在线列表
- 最新列表，获取最新10条新闻数据
- 排行榜，消费排行榜

## Redis 基础

### 下载安装安装

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

### 持久化之rdb 模式

开启rdb模式

```shell
# save 60 1000 //关闭RDB只需要将所有的save保存策略注释掉即可
```

还可以手动执行命令生成RDB快照，进入redis客户端执行命令**save**或**bgsave**可以生成dump.rdb文件， 每次命令执行都会将所有redis内存快照到一个新的rdb文件里，并覆盖原有rdb快照文件。

### 持久化之aof模式

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

### Jedis 链接Redis

引入jar包

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

JAVA 代码调用实例

```java
public class JedisSingleTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(20);
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);

        // timeout，这里既是连接超时又是读写超时，从Jedis 2.8开始有区分connectionTimeout和soTimeout的构造函数
        JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.0.60", 6379, 3000, null);
        Jedis jedis = null;
        try {
            //从redis连接池里拿出一个连接执行命令
            jedis = jedisPool.getResource();
            System.out.println(jedis.set("single", "zhuge"));
            System.out.println(jedis.get("single"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
            if (jedis != null)
                jedis.close();
        }
    }
}
```

### Redis 管道（Pipeline）使用

管道可以一次发送多个命令给Redis服务，减少网络开销，如果前面的命令执行失败，后面的命令会继续执行，不会被影响，最后把所有命令的执行结果一次返回。

```java
Pipeline pl = jedis.pipelined();
for (int i = 0; i < 10; i++) {
    pl.incr("pipelineKey");
    pl.set("zhuge" + i, "zhuge");
    //模拟管道报错
    // pl.setbit("zhuge", -1, true);
}
List<Object> results = pl.syncAndReturnAll();
System.out.println(results);
```

### Redis Lua 脚本使用

LUA 脚本可以代替Redis的事务执行，要么所有命令都执行成功，要么都失败，是原子性的，也可以减少网络开销。

在Redis控制台执行

```sh
127.0.0.1:6379> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

在代码中使用

```java
jedis.set("product_stock_10016", "15");  //初始化商品10016的库存
String script = " local count = redis.call('get', KEYS[1]) " +
                " local a = tonumber(count) " +
                " local b = tonumber(ARGV[1]) " +
                " if a >= b then " +
                "   redis.call('set', KEYS[1], a-b) " +
                "   return 1 " +
                " end " +
                " return 0 ";
Object obj = jedis.eval(script, Arrays.asList("product_stock_10016"), Arrays.asList("10"));
System.out.println(obj);
```

> 以上代码模拟减库存操作，如果后面执行失败，减掉的库存会回滚。

## Redis主从模式

![image-20210728183045747](https://raw.githubusercontent.com/telzhou618/images/main/img/image-20210728183045747.png)

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

sentinel哨兵是特殊的redis服务，不提供读写服务，主要用来监控redis实例节点。 哨兵架构下client端第一次从哨兵找出redis的主节点，后续就直接访问redis的主节点，不会每次都通过 sentinel代理访问redis的主节点，当redis的主节点发生变化，哨兵会第一时间感知到，并且将新的redis 主节点通知给client端(这里面redis的client端一般都实现了订阅功能，订阅sentinel发布的节点变动消息)

![image-20210728183140654](https://raw.githubusercontent.com/telzhou618/images/main/img/image-20210728183140654.png)

### **redis 哨兵架构搭建步骤**

```sh
1、复制一份sentinel.conf文件
cp sentinel.conf sentinel-26379.conf

2、将相关配置修改为如下值：
port 26379
daemonize yes
pidfile "/var/run/redis-sentinel-26379.pid"
logfile "26379.log"
dir "/usr/local/redis-5.0.3/data"
# sentinel monitor <master-redis-name> <master-redis-ip> <master-redis-port> <quorum>
# quorum是一个数字，指明当有多少个sentinel认为一个master失效时(值一般为：sentinel总数/2 + 1)，master才算真正失效
sentinel monitor mymaster 192.168.0.60 6379 2   # mymaster这个名字随便取，客户端访问时会用到

3、启动sentinel哨兵实例
src/redis-sentinel sentinel-26379.conf

4、查看sentinel的info信息
src/redis-cli -p 26379
127.0.0.1:26379>info
可以看到Sentinel的info里已经识别出了redis的主从

5、可以自己再配置两个sentinel，端口26380和26381，注意上述配置文件里的对应数字都要修改
```

### jedis 连接哨兵

```java
public class JedisSentinelTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);

        String masterName = "mymaster";
        Set<String> sentinels = new HashSet<String>();
        sentinels.add(new HostAndPort("192.168.0.60",26379).toString());
        sentinels.add(new HostAndPort("192.168.0.60",26380).toString());
        sentinels.add(new HostAndPort("192.168.0.60",26381).toString());
        //JedisSentinelPool其实本质跟JedisPool类似，都是与redis主节点建立的连接池
        //JedisSentinelPool并不是说与sentinel建立的连接池，而是通过sentinel发现redis主节点并与其建立连接
        JedisSentinelPool jedisSentinelPool = new JedisSentinelPool(masterName, sentinels, config, 3000, null);
        Jedis jedis = null;
        try {
            jedis = jedisSentinelPool.getResource();
            System.out.println(jedis.set("sentinel", "zhuge"));
            System.out.println(jedis.get("sentinel"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
            if (jedis != null)
                jedis.close();
        }
    }
}
```

### spring-boot 连接哨兵

- 引入依赖的jar包

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```

- 配置连接信息

```yaml
server:
  port: 8080

spring:
  redis:
    database: 0
    timeout: 3000
    sentinel:    #哨兵模式
      master: mymaster #主服务器所在集群名称
     nodes: 192.168.0.60:26379,192.168.0.60:26380,192.168.0.60:26381
   lettuce:
      pool:
        max-idle: 50
        min-idle: 10
        max-active: 100
        max-wait: 1000
```

- 使用代码

```java
@RestController
public class IndexController {

    private static final Logger logger = LoggerFactory.getLogger(IndexController.class);

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 测试节点挂了哨兵重新选举新的master节点，客户端是否能动态感知到
     * 新的master选举出来后，哨兵会把消息发布出去，客户端实际上是实现了一个消息监听机制，
     * 当哨兵把新master的消息发布出去，客户端会立马感知到新master的信息，从而动态切换访问的masterip
     *
     * @throws InterruptedException
     */
    @RequestMapping("/test_sentinel")
    public void testSentinel() throws InterruptedException {
        int i = 1;
        while (true){
            try {
                stringRedisTemplate.opsForValue().set("zhuge"+i, i+"");
                System.out.println("设置key："+ "zhuge" + i);
                i++;
                Thread.sleep(1000);
            }catch (Exception e){
                logger.error("错误：", e);
            }
        }
    }
}
```

## Redis 集群模式

Redis 集群至少需要3个主节点，一般需要给每个主节点配一个从节点。

![image-20210728183234826](https://raw.githubusercontent.com/telzhou618/images/main/img/image-20210728183234826.png)

### Redis集群搭建

```sh
第一步：在第一台机器的/usr/local下创建文件夹redis-cluster，然后在其下面分别创建2个文件夾如下
（1）mkdir -p /usr/local/redis-cluster
（2）mkdir 8001 8004

第一步：把之前的redis.conf配置文件copy到8001下，修改如下内容：
（1）daemonize yes
（2）port 8001（分别对每个机器的端口号进行设置）
（3）pidfile /var/run/redis_8001.pid  # 把pid进程号写入pidfile配置的文件
（4）dir /usr/local/redis-cluster/8001/（指定数据文件存放位置，必须要指定不同的目录位置，不然会丢失数据）
（5）cluster-enabled yes（启动集群模式）
（6）cluster-config-file nodes-8001.conf（集群节点信息文件，这里800x最好和port对应上）
（7）cluster-node-timeout 10000
 (8)# bind 127.0.0.1（bind绑定的是自己机器网卡的ip，如果有多块网卡可以配多个ip，代表允许客户端通过机器的哪些网卡ip去访问，内网一般可以不配置bind，注释掉即可）
 (9)protected-mode  no   （关闭保护模式）
 (10)appendonly yes
如果要设置密码需要增加如下配置：
 (11)requirepass zhuge     (设置redis访问密码)
 (12)masterauth zhuge      (设置集群节点间访问密码，跟上面一致)

第三步：把修改后的配置文件，copy到8004，修改第2、3、4、6项里的端口号，可以用批量替换：
:%s/源字符串/目的字符串/g 

第四步：另外两台机器也需要做上面几步操作，第二台机器用8002和8005，第三台机器用8003和8006

第五步：分别启动6个redis实例，然后检查是否启动成功
（1）/usr/local/redis-5.0.3/src/redis-server /usr/local/redis-cluster/800*/redis.conf
（2）ps -ef | grep redis 查看是否启动成功
    
第六步：用redis-cli创建整个redis集群(redis5以前的版本集群是依靠ruby脚本redis-trib.rb实现)
# 下面命令里的1代表为每个创建的主服务器节点创建一个从服务器节点
# 执行这条命令需要确认三台机器之间的redis实例要能相互访问，可以先简单把所有机器防火墙关掉，如果不关闭防火墙则需要打开redis服务端口和集群节点gossip通信端口16379(默认是在redis端口号上加1W)
# 关闭防火墙
# systemctl stop firewalld # 临时关闭防火墙
# systemctl disable firewalld # 禁止开机启动
# 注意：下面这条创建集群的命令大家不要直接复制，里面的空格编码可能有问题导致创建集群不成功
（1）/usr/local/redis-5.0.3/src/redis-cli -a zhuge --cluster create --cluster-replicas 1 192.168.0.61:8001 192.168.0.62:8002 192.168.0.63:8003 192.168.0.61:8004 192.168.0.62:8005 192.168.0.63:8006 

第七步：验证集群：
（1）连接任意一个客户端即可：./redis-cli -c -h -p (-a访问服务端密码，-c表示集群模式，指定ip地址和端口号）
    如：/usr/local/redis-5.0.3/src/redis-cli -a zhuge -c -h 192.168.0.61 -p 800*
（2）进行验证： cluster info（查看集群信息）、cluster nodes（查看节点列表）
（3）进行数据操作验证
（4）关闭集群则需要逐个进行关闭，使用命令：
/usr/local/redis-5.0.3/src/redis-cli -a zhuge -c -h 192.168.0.60 -p 800* shutdown
```

### java 操作Redis集群

- 引入依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

- java 使用实例

```java
public class JedisClusterTest {
    public static void main(String[] args) throws IOException {

        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);

        Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8001));
        jedisClusterNode.add(new HostAndPort("192.168.0.62", 8002));
        jedisClusterNode.add(new HostAndPort("192.168.0.63", 8003));
        jedisClusterNode.add(new HostAndPort("192.168.0.61", 8004));
        jedisClusterNode.add(new HostAndPort("192.168.0.62", 8005));
        jedisClusterNode.add(new HostAndPort("192.168.0.63", 8006));

        JedisCluster jedisCluster = null;
        try {
            //connectionTimeout：指的是连接一个url的连接等待时间
            //soTimeout：指的是连接上一个url，获取response的返回等待时间
            jedisCluster = new JedisCluster(jedisClusterNode, 6000, 5000, 10, "zhuge", config);
            System.out.println(jedisCluster.set("cluster", "zhuge"));
            System.out.println(jedisCluster.get("cluster"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (jedisCluster != null)
                jedisCluster.close();
        }
    }
}
```

### Spring-boot 操作Redis集群

- 引入依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```

- 配置链接信息

```yaml
server:
  port: 8080

spring:
  redis:
    database: 0
    timeout: 3000
    password: zhuge
    cluster:
      nodes: 192.168.0.61:8001,192.168.0.62:8002,192.168.0.63:8003,192.168.0.61:8004,192.168.0.62:8005,192.168.0.63:8006
   lettuce:
      pool:
        max-idle: 50
        min-idle: 10
        max-active: 100
        max-wait: 1000
```

- 使用代码

```java
@RestController
public class IndexController {

    private static final Logger logger = LoggerFactory.getLogger(IndexController.class);

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/test_cluster")
    public void testCluster() throws InterruptedException {
       stringRedisTemplate.opsForValue().set("zhuge", "666");
       System.out.println(stringRedisTemplate.opsForValue().get("zhuge"));
    }
}
```

### Redis 集群原理分析

Redis Cluster 将所有数据划分为 16384 个 slots(槽位)，每个节点负责其中一部分槽位。槽位的信息存储于每个节点中。当 Redis Cluster 的客户端来连接集群时，它也会得到一份集群的槽位配置信息并将其缓存在客户端本地。这样当客户端要查找某个 key 时，可以直接定位到目标节点。同时因为槽位的信息可能会存在客户端与服务器不一致的情况，还需要纠正机制来实现槽位信息的校验调整。

槽位定位算法

Cluster 默认会对 key 值使用 crc16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行取模来得到具体槽位。HASH_SLOT = CRC16(key) mod 16384



## Redis 核心原理

## Redis 常见问题及解决

## Redis 参考
