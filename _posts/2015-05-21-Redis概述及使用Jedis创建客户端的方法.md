---
layout: post
title:  "Redis概述及使用Jedis创建客户端的方法"
keywords: "cache, redis,jedis"
description: "Redis是一个开源的key-value持久化产品，它通常被称为数据结构服务器，它的值可以是字符串（String）、哈希（Map）、列表（List）、集合（Sets）和有序集合（Sorted sets）等类型，可以在这些类型上面做一些原子操作"
category: redis
tags: [redis]
---
###概述
Redis是一个开源的key-value持久化产品，它通常被称为数据结构服务器，它的值可以是字符串（String）、哈希（Map）、列表（List）、集合（Sets）和有序集合（Sorted sets）等类型，可以在这些类型上面做一些原子操作，如：字符串追加、增加Hash里面的值、添加元素到列表、计算集合的交集，并集和差集，为了取得好的性能，Redis是一个内存型数据库，不限于此，Redis也可以把数据持久化到磁盘中，或者把数据操作指令追加了一个日志文件，把它用于持久化，也可以用Redis容易的搭建master-slave架构用于数据复制，其它让它像缓存的特性包括，简单的check-and-set机制，pub/sub和配置设置,Redis可以用大部分程序语言来操作：C、C++、C#、Java、Node.js、php、ruby等等，Redis是用ANSIC写的，可以运行在多数POSIX系统。
### Redis的五种数据类型
#### String类型
String是最基本的类型，而且String类型是二进制安全的，意思是redis的String可以包含任何数据，比如jgp图片或者序列化的对象，从内部实现来看其实string可以看作byte数组，最大上限是1G字节
#### 类型
hash是一个string类型的field和value的映射表。添加，删除操作都是O(1)（平均）。 hash特别适合用于存储对象。相对于将对象的每个字段存成单个string类型。将一个对象存储在hash类型中会占用更少的内存，并且可以更方便的存取整个对象。省内存的原因是新建一个hash对象时开始是用 zipmap（又称为 small hash）来存储的。这个 zipmap 其实并不是hashtable，但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销。尽管zipmap的添加，删除，查找都是 O(n)，但是由于一般对象的field 数量都不太多。所以使用zipmap也是很快的,也就是说添加删除平均还是O(1)。如果field 或者 value的大小超出一定限制后，redis会在内部自动将zipmap替换成正常的hash实现.这个限制可以在配置文件中指定
#### List类型
list是一个链表结构，可以理解为一个每个子元素都是 string 类型的双向链表。主要功
能是push、pop、获取一个范围的所有值等。操作中key理解为链表的名字
#### Set类型
set是无序集合，最大可以包含(2的 32 次方-1)个元素。set 的是通过 hash table 实现的，所以添加，删除，查找的复杂度都是 O(1)。hash table 会随着添加或者删除自动的调整大小。需要注意的是调整 hash table 大小时候需要同步（获取写锁）会阻塞其他读写操作。可能不久后就会改用跳表（skip list）来实现。跳表已经在 sorted sets 中使用了。关于 set 集合类型除了基本的添加删除操作，其它有用的操作还包含集合的取并集(union)，交集(intersection)，差集(difference)。通过这些操作可以很容易的实现 SNS 中的好友推荐和 blog 的 tag 功能
#### Sorted Set
sorted set是有序集合，它在set的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，会自动重新按新的值调整顺序。可以理解了有两列的 mysql表，一列存value，一列存顺序。操作中key理解为sorted set的名字
### Redis主从复制
#### 介绍
Redis支持将数据同步到多台从库上，这种特性对提高读取性能非常有益。master可以有多个slave。除了多个slave连到相同的master外，slave也可以连接其它slave形成图状结构。主从复制不会阻塞master。也就是说当一个或多个slave与master进行初次同步数据时，master可以继续处理客户端发来的请求。相反slave在初次同步数据时则会阻塞不能处理客户端的请求。
主从复制可以用来提高系统的可伸缩性,我们可以用多个slave 专门用于客户端的读请求，比如sort操作可以使用slave来处理。也可以用来做简单的数据冗余。可以在 master 禁用数据持久化，只需要注释掉 master 配置文件中的所有 save 配置，然后只在 slave 上配置数据持久化
#### 过程
当设置好 slave 服务器后，slave 会建立和 master 的连接，然后发送 sync命令。无论是第一次同步建立的连接还是连接断开后的重新连接，master 都会启动一个后台进程，将数据库快照保存到文件中，同时 master 主进程会开始收集新的写命令并缓存起来。后台进程完成写文件后，master 就发送文件给 slave，slave 将文件保存到磁盘上，然后加载到内存恢复数据库快照到 slave 上。接着 master 就会把缓存的命令转发给 slave。而且后续 master 收到的写命令都会通过开始建立的连接发送给slave。从master到slave的同步数据的命令和从客户端发送的命令使用相同的协议格式。当 master 和 slave 的连接断开时 slave 可以自动重新建立连接。如果 master 同时收到多个 slave 发来的同步连接命令，只会启动一个进程来写数据库镜像，然后发送给所有 slave。
配置 slave服务器很简单，只需要在配置文件中加入如下配置
slaveof  192.168.1.1 6379     #指定 master的 ip 和端口。
### Redis持久化
通常Redis将数据存储在内存中或虚拟内存中，但它提供了数据持久化功能可以把内存中的数据持久化到磁盘。持久化有什么好处呢？比如可以保证断电后数据不会丢失，升级服务器也会变得更加方便。Redis提供了两种数据持久化的方式
#### RDB Snapshotting方式持久化（默认方式）
这种方式就是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为 dump.rdb。客户端也可以使用save或者bgsave命令通知redis做一次快照持久化。save操作是在主线程中保存快照的，由于redis是用一个主线程来处理所有客户端的请求，这种方式会阻塞所有客户端请求。所以不推荐使用。另一点需要注意的是，每次快照持久化都是将内存数据完整写入到磁盘一次，并不是增量的只同步增量数据。如果数据量大的话，写操作会比较多，必然会引起大量的磁盘IO操作，可能会严重影响性能。
这种方式的缺点也是显而易见的，由于快照方式是在一定间隔时间做一次的，所以如果 redis 意外当机的话，就会丢失最后一次快照后的所有数据修改
#### AOF方式持久化
这种方式 redis 会将每一个收到的写命令都通过 write 函数追加到文件中(默认 appendonly.aof)。当redis重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。当然由于操作系统会在内核中缓存write做的修改，所以可能不是立即写到磁盘上。这样的持久化还是有可能会丢失部分修改。不过我们可以通过配置文件告诉 redis我们想要通过fsync函数强制操作系统写入到磁盘的时机。有三种方式如下（默认是：每秒fsync一次）<br>
appendonly  yes     //启用日志追加持久化方式 <br>

*  appendfsync always     //每次收到写命令就立即强制写入磁盘，最慢的，但是保证完全持久化，不推荐使用。
* appendfsync everysec //每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，推荐使用。
* appendfsync no    //完全依赖操作系统，性能最好,持久化没保证。

日志追加方式同时带来了另一个问题。持久化文件会变的越来越大。例如我们调用 incr test 命令 100 次，文件中必须保存全部 100 条命令，其实有 99 条都是多余的。因为要恢复数据库状态其实文件中保存一条 set test 100 就够了。为了压缩这种持久化方式的日志文件。 redis 提供了 bgrewriteaof 命令。收到此命令 redis 将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的持久化日志文件
### Redis虚拟内存
#### 介绍
首先说明下redis的虚拟内存与操作系统虚拟内存不是一码事，但是思路和目的都是相同的。就是暂时把不经常访问的数据从内存交换到磁盘中，从而腾出宝贵的内存空间。对于 redis这样的内存数据库，内存总是不够用的。除了可以将数据分割到多个redis 服务器以外。另外的能够提高数据库容量的办法就是使用虚拟内存技术把那些不经常访问的数据交换到磁盘上。如果我们存储的数据总是有少部分数据被经常访问，大部分数据很少被访问，对于网站来说确实总是只有少量用户经常活跃。当少量数据被经常访问时，使用虚拟内存不但能提高单台redis数据库服务器的容量，而且也不会对性能造成太多影响。<br>
redis没有使用操作系统提供的虚拟内存机制而是自己在用户态实现了自己的虚拟内存机制。主要的理由有以下两点，操作系统的虚拟内存是以4k/页为最小单位进行交换的。而redis的大多数对象都远小于4k，所以一个操作系统页上可能有多个redis对象。另外redis的集合对象类型如 list,set可能存在于多个操作系统页上。最终可能造成只有10%的key被经常访问，但是所有操作系统页都会被操作系统认为是活跃的，这样只有内存真正耗尽时操作系统才会进行页的交换。相比操作系统的交换方式。redis可以将被交换到磁盘的对象进行压缩,保存到磁盘的对象可以去除指针和对象元数据信息。一般压缩后的对象会比内存中的对象小 10倍。这样redis的虚拟内存会比操作系统的虚拟内存少做很多IO操作。
#### 虚拟内存相关设置
 * vm-enabled  yes    开启虚拟内存功能
 * vm-swap-file  /tmp/redis.swap     交换出来value保存的文件路径/tmp/ redis.swapvm-max-memory268435456  redis使用的最大内存上限（256MB），超过上限后 redis开始交换value到磁盘swap文件中。建议设置为系统空闲内存的60 %-80%
 * vm-page-size  32  每个redis页的大小32个字节
* vm-pages  134217728  最多在文件中使用多少个页,交换文件的大小 =（vm-page-size* vm-pages）4 GB
* vm-max-threads  8 用于执行value对象换入换出的工作线程数量。0 表示不使用工作线程

### 环境安装
#### 下载、解压、安装

```
$ wget http://download.redis.io/releases/redis-2.8.13.tar.gz
$ tar xzf redis-2.8.13.tar.gz
$ cd redis-2.8.13
$ make
```
#### 启动Redis实例

```
$ src/redis-server
```
#### 通过内置客户端进行测试

```
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

### Redis配置文件

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
daemonize yes
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定pidfile/usr/local/redis/var/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字。
port 6379
4. 绑定的主机地址
bind 127.0.0.1
5. 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 0
6. 对客户端发送ACK信息，linux中单位为秒
tcp-keepalive 0
7. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为noticeloglevel notice
8. 日志记录位置，默认为标准输出
logfile/usr/local/redis/var/redis.log
9. 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id
databases 16
10. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
Save分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
Redis默认配置文件中提供了三个条件：
save 900 1
save 300 10
save 60 10000
11. 持久化失败以后，redis是否停止
stop-writes-on-bgsave-erroryes
12. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
rdbcompression yes
rdbchecksum yes
13. 指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb
14. 指定本地数据库存放目录
dir /usr/local/redis/var
15. 设置当本机为slave服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
 slaveof
16. 当master服务设置了密码保护时，slave服务连接master的密码
 masterauth
17. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH 命令提供密码，默认关闭requirepass foobared
18. 设置同一时间最大客户端连接数，在 Redis2.4中，最大连接数是被直接硬编码在代码里面的，而在2.6版本中这个值变成可配置的。maxclients 的默认值是 10000，你也可以在 redis.conf 中对这个值进行修改。当然，这个值只是 Redis 一厢情愿的值，Redis 还会照顾到系统本身对进程使用的文件描述符数量的限制。在启动时 Redis 会检查系统的 soft limit，以查看打开文件描述符的个数上限。如果系统设置的数字，小于咱们希望的最大连接数加32，那么这个 maxclients 的设置将不起作用，Redis 会按系统要求的来设置这个值。（加32是因为 Redis 内部会使用最多32个文件描述符，所以连接能使用的相当于所有能用的描述符号减32）。当上面说的这种情况发生时（maxclients 设置后不起作用的情况），Redis 的启动过程中将会有相应的日志记录。比如下面命令希望设置最大客户端数量为10000，所以 Redis 需要 10000+32 个文件描述符，而系统的最大文件描述符号设置为10144，所以 Redis 只能将maxclients 设置为 10144 – 32 = 10112。
 maxclients 10000
19. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
 maxmemory
slave-serve-stale-data yes
slave-read-only yes
repl-disable-tcp-nodelay no
slave-priority 100
20. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为Redis本身同步数据文件是按上面slave条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
appendonly no
21. 指定更新日志文件名，默认为appendonly.aof
 appendfilename appendonly.aof
22. 指定更新日志条件，共有3个可选值：
no：表示等操作系统进行数据缓存同步到磁盘（快）
always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec
23. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
vm-enabled no
24. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
vm-swap-file /tmp/redis.swap
25. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。
vm-max-memory 0
26. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不确定，就使用默认值
vm-page-size32
27. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
vm-pages 134217728
28. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
vm-max-threads 4
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
29. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
30. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
activerehashing yes
client-output-buffer-limit normal 0 00
client-output-buffer-limit slave256mb 64mb 60
client-output-buffer-limit pubsub32mb 8mb 60
hz 10
31. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
具体请参考：[https://raw.githubusercontent.com/antirez/redis/2.6/redis.conf](https://raw.githubusercontent.com/antirez/redis/2.6/redis.conf)

### Redis客户端配置
#### redis.properties
```
#最大分配的对象数
redis.pool.maxActive=1024
#最大能够保持idel状态的对象数
redis.pool.maxIdle=200
#当池内没有返回对象时，最大等待时间
redis.pool.maxWait=1000
#超时时间
redis.pool.timeout=1000
#当调用borrow Object方法时，是否进行有效性检查
redis.pool.0nBorrow=true
#redis部署的主机IP
redis.master.host=127.0.0.1
#redis端口号
redis.master.port=6739
```
Redis默认端口为6739，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
#### 对象池配置 

```
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<property name="maxActive" value="${redis.pool.maxActive}" />
	<property name="maxIdle" value="${redis.pool.maxIdle}" />
	<property name="maxWait" value="${redis.pool.maxWait}" />
    <property name="testOnBorrow" value="${redis.pool.onBorrow}">
</bean>
```

#### Jedis原生实现的配置

```
<bean id="cacheService" class="com.test.cache.CacheServiceImpl" init-method="init" destroy-method="destroy">
	<property name="masterHost" value="${redis.master.host}"/>
	<property name="masterPort" value="${reids.master.port}"/>
	<property name="timeOut" value="${redis.pool.timeout}" />
    <property name="poolConfig" ref="jedisPoolConfig" />
</bean>
```
#### 示例代码
* Redis客户端实现是Jedis，这是比较成熟的Redis客户端，也是官网推荐的，因为Redis 3.0之前的版本不支持集群功能，如果想在集群中启动多个独立的实例，在客户端通过“一致性hash”技术来实现对集群中各个Redis实例进行读写，Jedis通过ShardJedis对象对"一致性hash"提供支持

```
import com.test.cache.service
/**
 *缓存的Service接口
 */
public interface CacheService{

	public void set(String key, String value);

    public void set(String key, String value, int expireSeconds);

    public String get(String key);
    //递增,将key中储存的数字值增一
    public long incr(String key);

    public void expire(String key, int expireSeconds);

    public void del(String key);
    //递减,将key中存储的数字值减一
    public long decr(String key);
}

/**
 *缓存的接口的实现
 */
import com.test.cache.CacheServiceImpl
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

 public class CacheServiceImpl implements CacheService {
    private static Logger   logger= LoggerFactory.getLogger(CacheServiceImpl.class);
    
    private String             masterHost;
    private int                masterPort;
    private int                timeOut

    private volatile JedisPool jedisPool;

    private JedisPoolConfig    poolConfig;

    public void init() {
        jedisPool = new JedisPool(poolConfig, masterHost, masterPort,timeOut);
    }

    public void destroy() {
        getJedisPool().destroy();
        logger.info(String.format("destroy redis connection pool,  %s:%s", masterHost, masterPort));
    }


    @Override
    public void set(String key, String value) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();

        try {
            jedis.set(key, value);
            pool.returnResource(jedis);

        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }

    @Override
    public void set(String key, String value, int expireSeconds) {

        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            jedis.set(key, value);
            jedis.expire(key, expireSeconds);
            pool.returnResource(jedis);

        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }

    @Override
    public String get(String key) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            String result = jedis.get(key);
            pool.returnResource(jedis);
            return result;

        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }

    @Override
    public long incr(String key) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            long result = jedis.incr(key);
            pool.returnResource(jedis);
            return result;
        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }
    
    @Override
    public long decr(String key) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            long result = jedis.decr(key);
            pool.returnResource(jedis);
            return result;
        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }

    @Override
    public void expire(String key, int expireSeconds) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            jedis.expire(key, expireSeconds);
            pool.returnResource(jedis);

        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }

    }

    @Override
    public void del(String key) {
        JedisPool pool = getJedisPool();
        Jedis jedis = pool.getResource();
        try {
            jedis.del(key);
            pool.returnResource(jedis);
        } catch (Exception e) {
            pool.returnBrokenResource(jedis);
            throw new CacheException(e);
        }
    }

    public String getMasterHost() {
        return masterHost;
    }

    public void setMasterHost(String masterHost) {
        this.masterHost = masterHost;
    }

    public int getMasterPort() {
        return masterPort;
    }

    public void setMasterPort(int masterPort) {
        this.masterPort = masterPort;
    }

    public JedisPoolConfig getPoolConfig() {
        return poolConfig;
    }

    public void setPoolConfig(JedisPoolConfig poolConfig) {
        this.poolConfig = poolConfig;
    }
    public void setTimeOut(int timeOut){
    	this.timeOut=timeOut;
    }
    public Int getTimeOut(){
    	return timeOut;
    }
      private JedisPool getJedisPool() {
        return jedisPool;
    }
}
```
### 其他参考资料
* 所有特性请参考：[http://redis.io/documentation](http://redis.io/documentation)
* 所有命令请参考：[http://redis.io/commands#sorted_set](http://redis.io/commands#sorted_set)
