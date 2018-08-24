redis
=============

    1. 基础知识
    2. redis 的应用场景 
    3. redis 安装和部署（操作命令和步骤）
    4. redis分布式集群部署（操作命令和步骤）
    5. redis 分布式锁实现（包含代码实现）
    6. redis 分布式事务（包含代码实现） 
    7. redis 实现消息队列（包含代码实现）
    8. redis 实现秒杀（包含代码实现）
    9. redis FAQ
    10. redis 工作中常见问题和解决方案
    11. redis 大数据下实践
    12. redis 源码分析

#### 1. 基础知识
 redis是用C语言开发的一个开源的高性能键值对（key-value）数据库。它通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止redis支持的键值数据类型如下
字符串、列表（lists）、集合（sets）、有序集合（sorts sets）、哈希表（hashs）
#### 2. redis的应用场景
 缓存（数据查询、短连接、新闻内容、商品内容等等）。（最多使用）
 分布式集群架构中的session分离(会话缓存)。
 聊天室的在线好友列表。
 任务队列。（秒杀、抢购、12306等等）
 应用排行榜。
 网站访问统计。
 数据过期处理（可以精确到毫秒）
 存储时间戳（类似排行榜，使用redis的zset用于存储时间戳，时间会不断变化。例如，按照用户关注用户的最新动态列表。）


### 3. redis 安装和部署
下面介绍在Linux环境下，Redis的安装与部署，使用redis-3.0稳定版,因为redis从3.0开始增加了集群功能。在后面我也会分享redis集群。
 1.可以通过官网下载 地址：http://download.redis.io
 2.使用linux wget/yum/brew 命令 <br>
    
    wget http://download.redis.io/releases/redis-3.0.0.tar.gz

将redis-3.0.0.tar.gz拷贝到/usr/local下
    
    cp redis-3.0.0.rar.gz /usr/local
解压源码

    tar -zxvf redis-3.0.0.tar.gz 
进入解压后的目录进行编译

    cd /usr/local/redis-3.0.0
安装到指定目录 如 /usr/local/redis

    make PREFIX=/usr/local/redis install
redis.conf是redis的配置文件，redis.conf在redis源码目录。
拷贝配置文件到安装目录下
进入源码目录，里面有一份配置文件 redis.conf，然后将其拷贝到安装路径下

    cd /usr/local/redis
    mkdir conf
    cp /usr/local/redis-3.0.0/redis.conf  /usr/local/redis/bin
进入安装目录bin下

    cd /usr/local/redis/bin
此时我们看到的目录结构是这样的

    redis-benchmark redis性能测试工具
    redis-check-aof AOF文件修复工具
    redis-check-rdb RDB文件修复工具
    redis-cli redis命令行客户端
    redis.conf redis配置文件
    redis-sentinal redis集群管理工具
    redis-server redis服务进程

4.启动redis
 1.前端模式启动
直接运行bin/redis-server将以前端模式启动，前端模式启动的缺点是ssh命令窗口关闭则redis-server程序结束，不推荐使用此方法

    ./redis-server
如图


 2.后端模式启动
修改redis.conf配置文件， daemonize yes 以后端模式启动

    vim /usr/local/redis/bin/redis.conf

执行如下命令启动redis：

    cd /usr/local/redis
    ./bin/redis-server ./redis.conf
连接redis

    /usr/local/redis/bin/redis-cli 
5.关闭redis
强行终止redis进程可能会导致redis持久化数据丢失。正确停止Redis的方式应该是向Redis发送SHUTDOWN命令，命令为：

    cd /usr/local/redis
    ./bin/redis-cli shutdown
强行终止redis

    pkill redis-server
让redis开机自启

    vim /etc/rc.local
    //添加
    /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis-conf

至此redis已经全部安装完，
   
redis.conf 配置解析
- [解析配置文件redis.conf](https://blog.csdn.net/min996358312/article/details/59246275)
- [Redis笔记整理（一）：Redis安装配置与数据类型操作](http://blog.51cto.com/xpleaf/2091288)    
### 4. redis 分布式集群部署
>  1. 部署原理：
>  2. 实践

- [Redis3.0.7集群部署](https://www.jianshu.com/p/7cfcee0f0980)
- [Redis笔记整理（二）：Java API使用与Redis分布式集群环境搭建](http://blog.51cto.com/xpleaf/2091534)
- [分布式缓存利器之Redis集群安装与配置](https://www.jianshu.com/p/3fcb8777cb42)
- [史上最全 Redis 高可用技术解决方案大全](https://segmentfault.com/a/1190000014374473)
- [分布式缓存 Redis 使用心得](https://juejin.im/entry/56baa0cfc4c97100522945d3)
### 5. redis 分布式锁
1. 实现方案：

        public class RedisTool {
        private static final String LOCK_SUCCESS = "OK";
        private static final String SET_IF_NOT_EXIST = "NX";
        private static final String SET_WITH_EXPIRE_TIME = "PX";
    
        /**
         * 尝试获取分布式锁
         * @param jedis Redis客户端
         * @param lockKey 锁
         * @param requestId 请求标识
         * @param expireTime 超期时间
         * @return 是否获取成功
         */
        public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
    
            String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
    
            if (LOCK_SUCCESS.equals(result)) {
                return true;
            }
            return false;
    
            }
    
        }

可以看到，我们加锁就一行代码：jedis.set(String key, String value, String nxxx, String expx, int time)，这个set()方法一共有五个形参：<br>
第一个为key，我们使用key来当锁，因为key是唯一的。<br>
<br>第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。
<br>第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；
<br>第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。
<br>第五个为time，与第四个参数相呼应，代表key的过期时间。
<br>总的来说，执行上面的set()方法就只会导致两种结果：1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。2. 已有锁存在，不做任何操作。
<br>心细的童鞋就会发现了，我们的加锁代码满足我们可靠性里描述的三个条件。首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性。其次，由于我们对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。最后，因为我们将value赋值为requestId，代表加锁的客户端请求标识，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。由于我们只考虑Redis单机部署的场景，所以容错性我们暂不考虑。



- [Redis分布式锁的正确实现方式（Java版）](https://wudashan.cn/2017/10/23/Redis-Distributed-Lock-Implement/)
### 6. redis 分布式事务
> 1. 实现方案：
> 2. 实践

### 7. redis 消息队列

- [Redis笔记整理（三）：进阶操作与高级部分](http://blog.51cto.com/xpleaf/2091636)
### 8. redis 秒杀


- [java 用redis如何处理电商平台，秒杀、抢购超卖](https://blog.csdn.net/u012116196/article/details/51782934)
- [京东抢购服务高并发实践](http://www.uml.org.cn/qiyezjjs/201608174.asp)
- [网站大规模并发处理方案：电商秒杀与抢购](https://www.awaimai.com/348.html)
- [Redis分布式锁解决抢购问题](https://segmentfault.com/a/1190000011421467)

### 9. redis FAQ


    1、 问：为什么说Redis是单线程的以及Redis为什么这么快
        答： 1、完全基于内存，
            2、数据结构简单，
            3、采用单线程，避免了不必要的上下文切换和竞争条件，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
            4、使用多路I/O复用模型，非阻塞IO；
            5、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，
        参考：见下面参考链接
        
    2、问：redis是单线程的 如何提高多核cpu的利用率
       答：CPU不太可能是Redis的瓶颈，一般内存和网络才有可能是。 例如使用Redis的管道（pipelining）在liunx系统上运行可以达到500K的RPS(requests per second) ，因此，如果您的应用程序主要使用O(N) 或者O(log(N)) 的 命令，他们几乎不需要使用什么CPU。
         然而，为了最大限度的使用CPU，可以在同一个服务器部署多个Redis的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的，
         所以，如果你想使用多个CPU，你可以考虑一下分片（shard） 。。
         在Redis的客户端类库里面，比如RB（Ruby的客户端）和Predis（最常用的PHP客户端之一），能够使用一致性哈希（consistent hashing）来处理多个Redis实例。
       参考：见下面参考链接   
    3、问： mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据   
       答：
         相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。redis 提供 6种数据淘汰策略：
         voltile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
         volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
         volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
         allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
         allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
         noenviction（驱逐）：禁止驱逐数据
         redis 确定驱逐某个键值对后，会删除这个数据并，并将这个数据变更消息发布到本地（AOF 持久化）和从机（主从连接）
         详情（配置）可看下面的参考（深入理解Redis数据淘汰策略） 链接：https://blog.csdn.net/wtyvhreal/article/details/46390065
         
    4、问：Redis 常见的性能问题都有哪些？如何解决？
       答：
       Master写内存快照，save命令调度rdbSave函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务，所以Master最好不要写内存快照。
       Master
       AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响Master重启的恢复速度。Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化,如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。
       Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。
       Redis主从复制的性能问题，为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内
       
     5、问：redis与Mysql的如何保持数据一致性
        答：解决方法是 2PC或是Paxos协议，代价较大。
          所以我们采用的方式是：
          写数据只写db
          更新数据先更新db，再失效cache
          读数据，先读cache，未命中读db，写入cache
     6、问：Redis如何做到主从同步的，如果同步时候出现网络异常怎么办
        答：
        当Slave需要和Master进行数据同步时：
        1)     Salve会发送sync命令到Master
        2)     Master启动一个后台进程，将Redis中的数据快照保存到文件中
        3)     启动后台进程的同时，Master会将保存数据快照期间接收到的写命令缓存起来
        4)     Master完成写文件操作后，将该文件发送给Salve
        5)     Salve将文件保存到磁盘上，然后加载文件到内存恢复数据快照到Salve的Redis上
        6)     当Salve完成数据快照的恢复后，Master将这期间收集的写命令发送给Salve端
        7)     后续Master收集到的写命令都会通过之前建立的连接，增量发送给salve端
        总结一下，主从刚刚连接的时候，进行全量同步；全同步结束后，进行增量同步。当然，如果有需要，slave 在任何时候都可以发起全量同步
参考: <br>

- [为什么说Redis是单线程的以及Redis为什么这么快](https://zhuanlan.zhihu.com/p/34438275)
- [Redis FAQ](http://www.redis.cn/topics/faq.html)
- [Redis 常见问题 ](https://www.jianshu.com/p/b8b6b7063cf0)
- [深入理解Redis数据淘汰策略](https://blog.csdn.net/wtyvhreal/article/details/46390065)
- [关于Redis的一些思考和总结](https://www.jianshu.com/p/b8880db58241)
- [史上最全 50 道 Redis 面试题及答案](https://juejin.im/post/5b7151e951882561392806e0)
- [从头到尾解析Hash表算法](https://blog.csdn.net/v_JULY_v/article/details/6256463)
- [Redis持久化详解（RDB、AOF）](https://www.jianshu.com/p/5ce3c0cb66e4)
- [Redis持久化----RDB和AOF 的区别](https://blog.csdn.net/ljheee/article/details/76284082)
- [Redis持久化RDB和AOF](https://my.oschina.net/davehe/blog/174662)
- [Redis主从同步：全量同步 增量同步](https://blog.csdn.net/u012538947/article/details/80601356)
- [Redis主从同步原理-PSYNC](https://blog.csdn.net/sk199048/article/details/77922589)
- [解析配置文件 redis.conf、Redis持久化RDB、Redis的主从复制](https://www.jianshu.com/p/fc87cedce780)
- [构建高性能数据库缓存之redis主从复制](http://blog.51cto.com/cfwlxf/1433637)
- [redis的相关操作以及相关环境的搭建与连接（哨兵环境、cluster环境等](https://github.com/youzhibing/redis)
### 10. redis 工作中常见问题和解决方案
        1. 问：如何做到redis和数据库同步

- [Redis与Mysql的数据一致性](https://cdn2.jianshu.io/p/23abe7620096?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
        
        2. 问：redis是如何实现主备方案的

- [redis实现主从复制和高可用(主从切换)](https://my.oschina.net/manmao/blog/550428)
        
### 11. redis 大数据下实践

### 12. redis 源码分析