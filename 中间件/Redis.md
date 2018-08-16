redis
=============

相关技术网站：<br>
   - [史上最全 50 道 Redis 面试题及答案](https://juejin.im/post/5b7151e951882561392806e0)

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

    默认的配置 redis.conf 文件中，首先约定了存储单位：
    1k => 1000 bytes
    1kb => 1024 bytes
    1m => 1000000 bytes
    1mb => 10241024 bytes
    1g => 1000000000 bytes
    1gb => 10241024*1024 bytes
    Redis 配置中对单位的大小写不敏感，1GB、1Gb和1gB都是相同的。由此也说明，Redis 只支持 bytes，不支持 bit 单位。
    Redis 支持以 “includes” 的方式引入其他配置文件，比如：
    include/path/to/local.conf
    include/path/to/other.conf
    需要注意的是，假如多个一个配置项在不同配置文件中都有定义，则以最后一行读入的为准，就是说后面的配置项会覆盖前面的配置项。
    
    1.1通用配置
    默认情况下，Redis 并不是以 daemon 形式来运行的。通过 daemonize 配置项可以控制Redis的运行形式，如果改为 yes，那么 Redis 就会以 daemon 形式运行：
    daemonize no
    当以 daemon 形式运行时，Redis 会生成一个 pid 文件，默认会生成在 /var/run/Redis.pid 。当然，可以通过 pidfile 来指定 pid 文件生成的位置，比如：
    pidfile /path/to/Redis.pid
    默认情况下，Redis 会响应本机所有可用网卡的连接请求。当然，Redis 允许通过 bind 配置项来指定要绑定的IP，比如：
    bind 192.168.1.2 10.8.4.2
    Redis的默认服务端口是6379，可以通过 port 配置项来修改。如果端口设置为0的话，Redis 便不会监听端口了。
    port 6379
    可是，如果Redis不监听端口，还怎么与外界通信呢？其实Redis还支持通过 unix socket 方式来接收请求。可以通过 unix socket 配置项来指定 unix socket 文件的路径，并通过 unix socket perm 来指定文件的权限。
    unixsocket /tmp/Redis.sock
    unixsocketperm 700
    
    在高 QPS 的环境下需要提高 backlog 的值来避免 TCP 的慢连接问题。想要提高 backlog 的值，除了需要设置 Redis 的 tcp-backlog ，还要同时提高更改 Linux 的配置，否则，Linux内核会默认将其截取为 /proc/sys/net/core/somaxconn 的大小。
    tcp-backlog 511
    当一个 Redis-client 一直没有请求发向 server 端，那么 server 端有权主动关闭这个连接，可以通过timeout 来设置“空闲超时时限”，0表示永不关闭。
    timeout 0
    TCP连接保活策略，可以通过 tcp-keepalive 配置项来进行设置，单位为秒，假如设置为60秒，则 server 端会每60秒向连接空闲的客户端发起一次 ACK 请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测。
    tcp-keepalive 0
    Redis支持通过 loglevel 配置项设置日志等级，共分四级，即 debug、verbose、notice、warning。
    loglevel notice
    Redis也支持通过 logfile 配置项来设置日志文件的生成位置。如果设置为空字符串，则 Redis 会将日志输出到标准输出。假如在 daemon 情况下将日志设置为输出到标准输出，则日志会被写到 /dev/null 中。
    logfile ""
    如果希望日志打印到 syslog 中，也很容易，通过 syslog-enabled 来控制。另外，syslog-ident 还可以指定syslog 里的日志标志，比如：
    syslog-ident Redis
    而且还支持指定 syslog 设备，值可以是 USER 或 LOCAL0-LOCAL7 。具体可以参考 syslog 服务本身的用法。
    syslog-facility local0
    对于 Redis 来说，可以设置其数据库的总数量，假如希望一个 Redis 包含16个数据库，那么设置如下：
    databases 16
    这16个数据库的编号将是0到15。默认的数据库是编号为0的数据库。用户可以使用 select <DBid> 来选择相应的数据库。
    
    1.2快照配置
    快照，主要涉及的是 Redis 的 RDB 持久化相关的配置。
    可以用如下的指令来让数据保存到磁盘上，即控制 RDB 快照功能：
    save <seconds> <changes>
    举例来说：
    save 900 1 //表示每15分钟且至少有1个 key 改变，就触发一次持久化
    save 300 10 //表示每5分钟且至少有10个 key 改变，就触发一次持久化
    save 60 10000 //表示每60秒至少有10000个 key 改变，就触发一次持久化
    如果想禁用 RDB 持久化的策略，只要不设置任何 save 指令就可以，或者给 save 传入一个空字符串参数也可以达到相同效果，就像这样：
    save""
    如果用户开启了 RDB 快照功能，那么在 Redis 持久化数据到磁盘时如果出现失败，默认情况下，Redis 会停止接受所有的写请求。这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。如果 Redis 不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。
    如果下一次 RDB 持久化成功，Redis 会自动恢复接受写请求。
    当然，如果不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话完全可以关闭这个功能，以便在快照写入失败时，也能确保 Redis 继续接受新的写请求。配置项如下：
    stop-writes-on-bgsave-error yes
    对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，Redis 会采用 LZF 算法进行压缩。如果不想消耗 CPU 来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。
    rdbcompression yes
    在存储快照后，我们还可以让 Redis 使用 CRC64 算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。
    rdbchecksum yes
    我们还可以设置快照文件的名称，默认是这样配置的：
    dbfilename dump.rdb
    还可以设置这个快照文件存放的路径。比如默认设置就是：
    dir ./db/
    
    1.3主从配置
    Redis 提供了主从同步功能。
    通过 slaveof 配置项可以控制某一个 Redis 作为另一个 Redis 的从服务器，通过指定 IP 和端口来定位到主Redis 的位置。一般情况下建议用户为从 Redis 设置一个不同频率的快照持久化的周期，或者为从 Redis 配置一个不同的服务端口等等。
    slaveof <masterip> <masterport>
    如果主 Redis 设置了验证密码的话（使用 requirepass 来设置），则在从 Redis 的配置中要使用masterauth 来设置校验密码，否则的话，主 Redis 会拒绝从 Redis 的访问请求。
    masterauth <master-password>
    当从 Redis 失去了与主 Redis 的连接，或者主从同步正在进行中时， Redis 该如何处理外部发来的访问请求呢？这里，从 Redis 可以有两种选择：
    第一种选择：如果 slave-serve-stale-data 设置为 yes （默认），则从 Redis 仍会继续响应客户端的读写请求。
    第二种选择：如果 slave-serve-stale-data 设置为 no，则从 Redis 会对客户端的请求返回 “SYNC with master inprogress” ，当然也有例外，当客户端发来 INFO 请求和 SLAVEOF 请求，从 Redis 还是会进行处理。
    可以控制一个从 Redis 是否可以接受写请求。将数据直接写入从 Redis ，一般只适用于那些生命周期非常短的数据，因为在主从同步时，这些临时数据就会被清理掉。自从 Redis2.6 版本之后，默认从 Redis 为只读。
    slave-read-only yes
    只读的从 Redis 并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用 rename-command指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：
    rename-command CONFIG b8c02d524045429941cc15f59e41cb7be6c52
    从 Redis 会周期性的向主 Redis 发出 PING 包。可以通过 repl_ping_slave_period 指令来控制其周期。默认是10秒。
    repl-ping-slave-period 10
    在主从同步时，可能在这些情况下会有超时发生：
    （1）以从 Redis 的角度来看，当有大规模 IO 传输时。
    （2）以从 Redis 的角度来看，当数据传输或 PING 时，主 Redis 超时
    （3）以主 Redis 的角度来看，在回复从 Redis 的 PING 时，从 Redis 超时
    用户可以设置上述超时的时限，不过要确保这个时限比 repl-ping-slave-period 的值要大，否则每次主Redis 都会认为从 Redis 超时。
    repl-timeout 60
    我们可以控制在主从同步时是否禁用 TCP_NODELAY 。如果开启 TCP_NODELAY ，那么主 Redis 会使用更少的 TCP 包和更少的带宽来向从 Redis 传输数据。但是这可能会增加一些同步的延迟，大概会达到40毫秒左右。如果关闭了 TCP_NODELAY ，那么数据同步的延迟时间会降低，但是会消耗更多的带宽。
    repl-disable-tcp-nodelay no
    我们还可以设置同步队列长度。队列长度（ backlog )是主 Redis 中的一个缓冲区，在与从 Redis 断开连接期间，主 Redis 会用这个缓冲区来缓存应该发给从 Redis 的数据。这样的话，当从 Redis 重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可。
    repl-backlog-size 1mb
    如果主 Redis 等了一段时间之后，还是无法连接到从 Redis ，那么缓冲队列中的数据将被清理掉。我们可以设置主 Redis 要等待的时间长度。如果设置为0，则表示永远不清理。默认是1个小时。
    repl-backlog-ttl 3600
    我们可以给众多的从 Redis 设置优先级，在主 Redis 持续工作不正常的情况，优先级高的从 Redis 将会升级为主 Redis 。而编号越小，优先级越高。比如一个主 Redis 有三个从 Redis ，优先级编号分别为10、100、25，那么编号为10的从 Redis 将会被首先选中升级为主 Redis 。当优先级被设置为0时，这个从Redis 将永远也不会被选中。默认的优先级为100。
    slave-priority 100
    假如主 Redis 发现有超过M个从 Redis 的连接延时大于N秒，那么主 Redis 就停止接受外来的写请求。这是因为从 Redis 一般会每秒钟都向主Redis发出PING，而主Redis会记录每一个从 Redis 最近一次发来 PING 的时间点，所以主 Redis 能够了解每一个从 Redis 的运行情况。
    min-slaves-to-write 3
    min-slaves-max-lag 10
    上面这个例子表示，假如有大于等于3个从 Redis 的连接延迟大于10秒，那么主 Redis 就不再接受外部的写请求。上述两个配置中有一个被置为0，则这个特性将被关闭。默认情况下 min-slaves-to-write 为0，而 min-slaves-max-lag 为10。
    
    1.4 安全配置
    我们可以要求 Redis 客户端在向 Redis-server 发送请求之前，先进行密码验证。由于 Redis 性能非常高，每秒钟可以完成多达15万次的密码尝试，所以最好设置一个足够复杂的密码，否则很容易被黑客破解。
    requirepass chenlongfei
    这里通过 requirepass 将密码设置成我的名字。
    Redis 允许我们对 Redis 指令进行更名，比如将一些比较危险的命令改个名字，避免被误执行。比如可以把 CONFIG 命令改成一个很复杂的名字，这样可以避免外部的调用，同时还可以满足内部调用的需要：
    rename-command CONFIG b840fc02d5240454299c15f59e41cb7be6c89
    我们甚至可以禁用掉 CONFIG 命令，那就是把 CONFIG 的名字改成一个空字符串：
    rename-command CONFIG ""
    但需要注意的是，如果使用 AOF 方式进行数据持久化，或者需要与从 Redis 进行通信，那么更改指令的名字可能会引起一些问题。
    
    1.5 限制配置
    我们可以设置 Redis 同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当无法设置进程文件句柄限制时， Redis 会设置为当前的文件句柄限制值减去32，因为 Redis 会为自身内部处理逻辑留一些句柄出来。
    如果达到了此限制， Redis 则会拒绝新的连接请求，并且向这些连接请求方发出 “max number of clients reached” 以作回应。
    maxclients 10000
    我们甚至可以设置 Redis 可以使用的内存量。一旦到达内存使用上限， Redis 将会试图移除内部数据，移除规则可以通过 maxmemory-policy 来指定。
    如果 Redis 无法根据移除规则来移除内存中的数据，或者我们设置了“不允许移除”，那么 Redis 则会针对那些需要申请内存的指令返回错误信息，比如 SET 、 LPUSH 等。但是对于无内存申请的指令，仍然会正常响应，比如 GET 等。
    maxmemory <bytes>
    需要注意的一点是，如果 Redis 是主 Redis （说明 Redis 有从 Redis ），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在设置的是“不移除”的情况下，才不用考虑这个因素。
    对于内存移除规则来说， Redis 提供了多达6种的移除规则。他们是：
    （1）volatile-lru：使用 LRU 算法移除过期集合中的 key
    （2）allkeys-lru：使用 LRU 算法移除 key
    （3）volatile-random：在过期集合中移除随机的 key
    （4）allkeys-random：移除随机的 key
    （5）volatile-ttl：移除那些 TTL 值最小的 key ，即那些最近才过期的 key
    （6）noeviction：不进行移除。针对写操作，只是返回错误信息
    无论使用上述哪一种移除规则，如果没有合适的 key 可以移除的话， Redis 都会针对写请求返回错误信息。
    maxmemory-policy volatile-lru
    LRU算法和最小TTL算法都并非是精确的算法，而是估算值。所以可以设置样本的大小。假如 Redis 默认会检查三个 key 并选择其中 LRU 的那个，那么可以改变这个 key 样本的数量。
    maxmemory-samples 3
    
    1.6 AOF配置
    默认情况下， Redis 会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的，但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。
    如上半部分中介绍的， AOF 是一种更好的保持数据一致性的方式。即使当服务器断电时，也仅会有1秒钟的写请求丢失，当 Redis 进程出现问题且操作系统运行正常时，甚至只会丢失一条写请求。
    官方建议， AOF 机制和 RDB 机制可以同时使用，不会有任何冲突。
    appendonly yes
    我们还可以设置 AOF 文件的名称：
    appendfilename "appendonly.aof"
    fsync()调用，用来告诉操作系统立即将缓存的指令写入磁盘。一些操作系统会“立即”进行，而另外一些操作系统则会“尽快”进行。
    Redis支持三种不同的模式：
    （1）no：不调用 fsync() 。而是让操作系统自行决定 sync 的时间。这种模式下， Redis 的性能会最快。
    （2）always：在每次写请求后都调用 fsync() 。这种模式下， Redis 会相对较慢，但数据最安全。
    （3）everysec：每秒钟调用一次 fsync() 。这是性能和安全的折衷。
    默认情况下为 everysec 。
    appendfsync everysec
    当 fsync 方式设置为 always 或 everysec 时，如果后台持久化进程需要执行一个很大的磁盘IO操作，那么 Redis 可能会在 fsync() 调用时卡住。目前尚未修复这个问题，这是因为即使我们在另一个新的线程中去执行 fsync() ，也会阻塞住同步写调用。
    为了缓解这个问题，我们可以使用下面的配置项，这样的话，当 BGSAVE 或 BGWRITEAOF 运行时， fsync() 在主进程中的调用会被阻止。这意味着当另一路进程正在对 AOF 文件进行重构时， Redis 的持久化功能就失效了，就好像我们设置了 “appendsync none” 一样。如果 Redis 有时延问题，那么可以将下面的选项设置为yes。否则请保持no，因为这是保证数据完整性的最安全的选择。
    no-appendfsync-on-rewrite no
    我们允许 Redis 自动重写 aof 。当 aof 增长到一定规模时， Redis 会隐式调用 BGREWRITEAOF 来重写log文件，以缩减文件体积。
    Redis 是这样工作的： Redis 会记录上次重写时的 aof 大小。假如 Redis 自启动至今还没有进行过重写，那么启动时 aof 文件的大小会被作为基准值。这个基准值会和当前的 aof 大小进行比较。如果当前 aof 大小超出所设置的增长比例，则会触发重写。另外还需要设置一个最小大小，是为了防止在 aof 很小时就触发重写。
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    如果设置 auto-aof-rewrite-percentage 为0，则会关闭此重写功能。
    指Redis在恢复时，会忽略最后一条可能存在问题的指令，默认值yes。即在 aof 写入时，可能存在指令写错的问题(突然断电，写了一半)，这种情况下，yes会log并继续，而no会直接恢复失败。
    aof-load-truncated yes
    
    1.7 LUA脚本配置
    lua 脚本的最大运行时间是需要被严格限制的，单位是毫秒：
    lua-time-limit 5000
    如果此值设置为0或负数，则既不会有报错也不会有时间限制。
    
    1.8 集群设置
    平常的 Redis 实例不能作为集群的节点，只有作为集群节点启动的实例才可以。下面的配置可以是 Redis 实例作为集群节点启动：
    cluster-enabled yes
    每个集群节点都有一个集群配置文件，该文件是由集群节点来创建和维护的，不能人工参与。每个集群节点需要不同的配置文件，所以需要保证同一个系统下的集群节点没有重名的配置文件，建议以端口号标记配置文件。
    cluster-config-file nodes-6379.conf
    当节点超时大于 cluster-node-timeout 的时候后，就会被认为宕机了，单位为毫秒。
    cluster-node-timeout 15000
    Redis 集群有一种 failover （故障转移）机制，即当主 Redis 宕机之后，会有一个最合适的从 Redis 充当主 Redis 。但是，当从 Redis 的数据“太老”了，与住 Redis 的标准数据偏差很大，为了保证数据一致性， Redis 会放弃 failover 。判别从 Redis 的的数据是不是“太老”有两种方法：
    （1）如果有多个从 Redis 可以接替主 Redis 的工作，则它们会交换信息，选取“最佳复制偏移”（接受了原主 Redis 最多的数据同步）的从 Redis 作为下一任主 Redis 。
    （2）每个从Redis计算与原主Redis最后一次数据同步的时间，当最短的时间间隔大于某个临界点的时候，集群则放弃failover。
    方法（2）当中的临界点可以通过配置调节，临界点的计算规则为：
    (node-timeout * slave-validity-factor)+ repl-ping-slave-period
    如node-timeout为30秒，slave-validity-factor为10秒，repl-ping-slave-period为10秒，当与原主Redis最后一次对话的时间间隔超过310秒的时候，集群就会放弃failover。
    当slave-validity-factor太大会使一台数据“太老”的从Redis充当主Redis；而slave-validity-factor太小可能会造成找不到合适的从Redis继任。
    默认的slave-validity-factor为10。
    cluster-slave-validity-factor 10
    考虑一种极端情况，集群有一台主Redis和四台从Redis，从Redis全部挂掉，failover机制有可能造成集群只有主Redis而无从Redis的尴尬境况。为了保证集群的名副其实，可以规定，当从Redis少于某个数量时，拒绝执行failover。
    cluster-migration-barrier 1
    默认情况下，当集群检测到某个哈希槽（hash slot）没有被覆盖（没有任何节点为此服务）会停止接受查询服务，如果集群部分宕机最终会导致整个集群不可用，当哈希槽重新被全覆盖的时候会自动变为可用。如果希望那些哈希槽被覆盖的集群节点继续接受查询服务，需要将cluster-require-full-coverage设置为no。
    cluster-require-full-coverage yes
    
    1.9 慢日志配置
    Redis慢日志是指一个系统进行日志查询超过了指定的时长。这个时长不包括IO操作，比如与客户端的交互、发送响应内容等，而仅包括实际执行查询命令的时间。
    针对慢日志可以设置两个参数，一个是执行时长，单位是微秒，另一个是慢日志的长度。当一个新的命令被写入日志时，最老的一条会从命令日志队列中被移除。单位是微秒，即1000000表示一秒。负数则会禁用慢日志功能，而0则表示强制记录每一个命令。
    slowlog-log-slower-than 10000
    慢日志最大长度，可以随便填写数值，没有上限，但要注意它会消耗内存。可以使用SLOWLOG RESET来重设这个值。
    slowlog-max-len 128
    
    1.10 延迟监控配置
    Redis的延迟监控子系统会在运行时对不同操作取样，以此来收集与延迟相关的数据，这些信息可以通过LATENCY命令以报表的形式呈现给用户。
    系统只会记录那些执行时间等于或大于atency-monitor-threshold的操作，该值默认为0，代表关闭监控，因为收集延迟数据多少会影响Redis的性能。
    latency-monitor-threshold 0
    
    1.11事件通知配置
    Redis可以向客户端通知某些事件的发生。
    例如，键空间（keyspace）时间通知如果开启，一个客户端对Database 0中的“foo”键执行了DEL操作，两条信息会通过Pub/Sub发布出去：
    PUBLISH__keyspace@0__:foo del
    PUBLISH__keyevent@0__:del foo
    可以选择需要发送哪种类型的通知，每种类型用一个字母代表：
    K 键空间事件,发布到“keyspace@<db> prefix”频道
    E 键事件, 发布到“ keyevent@<db> prefix”频道
    g 通用事件，比如 DEL,EXPIRE, RENAME, ...等操作都属于
    $ String操作
    l List操作
    s Set操作
    h Hash操作
    z Sorted set操作
    x 过期操作
    e 驱逐操作（因为内存不足数据被删除）
    A 代表“g$lshzxe”的组合, 所以“AKE”可以代表所有事件
    
    notify-keyspace-events配置以上述的字母组合为参数，举例说明：
    （1）notify-keyspace-events Elg
    当有List操作或通用操作，发布通知到“ keyevent@<db> prefix”频道
    （2）notify-keyspace-events Ex
    当有键的过期操作时，发布通知到“keyevent@0:expired”频道
    默认情况下，notify-keyspace-events的参数为空字符串，代表关闭通知。
    notify-keyspace-events ""
    
    1.12 高级配置
    Hash 在条目数量较小的时候会使用一种高效的内存数据结构编码，当超过某个临界点就会采用另一种存储方式，该临界点由下面的两个配置决定：
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    与Hash类似，较小的List会以一种特殊的编码方式来节省空间，只要List不超过下面的上限：
    list-max-ziplist-entries 512
    list-max-ziplist-value 64
    Set只有在满足下面的条件时才会采用特殊编码方式：Set中存储的恰好都是十进制的整数，而且长度不超过64位（有符号）。数量上限为：
    set-max-intset-entries 512
    同样，有序集合也会采用特殊编码来节省空间，只要不超过上限：
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    RedisHyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定并且很小的。当HyperLogLog用稀疏式表示法时所用内存超过下面的限制，就会转换成稠密式表示，为了更高的内存利用率，官方建议值为3000。
    hll-sparse-max-bytes 3000
    Redis 在每 100 毫秒时使用 1 毫秒的 CPU时间来对 Redis 的 hash 表进行重新 hash 。当使用场景中有非常严格的实时性需要，不能够接受 Redis 时不时的对请求有 2 毫秒的延迟的话，把这项配置为 no 。
    如果没有这么严格的实时性要求，可以设置为 yes ，以便能够尽可能快的释放内存。
    activerehashing yes
    客户端的输出缓冲区的限制，因为某种原因客户端从服务器读取数据的速度不够快，可用于强制断开连接（一个常见的原因是一个发布 / 订阅客户端消费消息的速度无法赶上生产它们的速度）。
    可以三种不同客户端的方式进行设置：
    （1）normal -> 正常客户端
    （2）slave -> slave 和 MONITOR 客户端
    （3）pubsub -> 至少订阅了一个 pubsub channel 或 pattern 的客户端
    每个client-output-buffer-limit 语法 :
    client-output-buffer-limit<class> <hard limit> <soft limit> <soft seconds> 一旦达到硬限制客户端会立即断开，或者达到软限制并保持达成的指定秒数（连续）。
    例如，如果硬限制为 32 兆字节和软限制为 16 兆字节 /10 秒，如果输出缓冲区的大小达到 32 兆字节，客户端将会立即断开，客户端达到 16 兆字节和连续超过了限制 10 秒，也将断开连接。
    默认 normal 客户端不做限制，因为他们在一个请求后未要求时（以推的方式）不接收数据，只有异步客户端可能会出现请求数据的速度比它可以读取的速度快的场景。
    把硬限制和软限制都设置为 0 来禁用该特性
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    Redis 会按照一定的频率来执行后台任务，比如关闭超时的客户端，清除过期键等。不是所有的任务都会按照相同的频率来执行，但 Redis 依照指定的“ Hz ”值来执行检查任务。
    hz 10
    aof rewrite 过程中，是否采取增量文件同步策略，默认为“yes”。 rewrite 过程中,每32M数据进行一次文件同步，这样可以减少 aof 大文件写入对磁盘的操作次数。
    aof-rewrite-incremental-fsync yes
    
- [Redis笔记整理（一）：Redis安装配置与数据类型操作](http://blog.51cto.com/xpleaf/2091288)    

### 4. redis 分布式集群部署
>  1. 部署原理：
>  2. 实践

- [Redis3.0.7集群部署](https://www.jianshu.com/p/7cfcee0f0980)
- [Redis笔记整理（二）：Java API使用与Redis分布式集群环境搭建](http://blog.51cto.com/xpleaf/2091534)
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

可以看到，我们加锁就一行代码：jedis.set(String key, String value, String nxxx, String expx, int time)，这个set()方法一共有五个形参：
第一个为key，我们使用key来当锁，因为key是唯一的。
第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。
第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；
第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。
第五个为time，与第四个参数相呼应，代表key的过期时间。
总的来说，执行上面的set()方法就只会导致两种结果：1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。2. 已有锁存在，不做任何操作。
心细的童鞋就会发现了，我们的加锁代码满足我们可靠性里描述的三个条件。首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性。其次，由于我们对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。最后，因为我们将value赋值为requestId，代表加锁的客户端请求标识，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。由于我们只考虑Redis单机部署的场景，所以容错性我们暂不考虑。



- [Redis分布式锁的正确实现方式（Java版）](https://wudashan.cn/2017/10/23/Redis-Distributed-Lock-Implement/)
### 6. redis 分布式事务
> 1. 实现方案：
> 2. 实践

### 7. redis 消息队列

- [Redis笔记整理（三）：进阶操作与高级部分](http://blog.51cto.com/xpleaf/2091636)
### 8. redis 秒杀

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
-[redis持久化RDB和AOF](https://my.oschina.net/davehe/blog/174662)
### 10. redis 工作中常见问题和解决方案
        1. 问：如何做到redis和数据库同步
        
        2. 问：redis是如何实现主备方案的

- [redis实现主从复制和高可用(主从切换)](https://my.oschina.net/manmao/blog/550428)
        
### 11. redis 大数据下实践
### 12. redis 源码分析