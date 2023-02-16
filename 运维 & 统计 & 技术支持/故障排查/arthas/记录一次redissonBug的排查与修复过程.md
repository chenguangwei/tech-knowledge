## 记录一次redisson Bug的排查与修复过程



朴朴科技java项目连接redis使用的是流行的redis驱动项目redisson,使用的是sentinel模式,但是在某次aws的内部网络出现抖动的时候,发现网络抖动结束之后,有部分redis请求依旧响应缓慢.下层的redis请求缓慢导致上层的tomcat io线程大量阻塞,最终服务无法响应请求.

拥有的信息如下

> 1) 经过aws工作人员核实,网络确实仅仅只有短暂的抖动(10多分钟),后续完全恢复.
>
> 2) 根据zipkin的监控,发现在在网络波动结束后,依旧有大量耗时大于1秒的redis请求响应.并且持续未能完全恢复.

猜测:是否redisson在网络抖动恢复后,依旧影响后续的请求.



02

—

阶段1：盲目重试，尝试复现



盲目重试,但是由于缺乏有效的观测指标,以及对redisson内部机制不了解,以及问题本身并不是全量请求都出现问题,所以这个阶段的压测最终只能是做了无用功





03

—

### 阶段2: 阅读redisson源码,定位导致阻塞的调用点



开始看redisson的请求响应部分流程的代码,标记出有可能造成响应缓慢,阻塞响应的位置,最后结合代码,标记出几点

> 1) 获取空闲连接时,没有空闲连接,且达到连接上限,或者无空闲连接,但是未达上限,创建新可用连接的步骤
>
> 2) 发送redis命令,并且等待netty的`writefuture`确认redis指令发送成功
>
> 3) 发送成功后,等待netty事件循环获取对应响应,释放信号
>
> 4) 当1,2,3阶段失败,触发超时器执行重试



> 备注:对源码感兴趣的同学可以跟踪redisson 3.11.0中的
>
> 1) 从连接池获取连接的过程: `org.redisson.connection.pool.ConnectionPool#acquireConnection` 方法
>
> 2) 发送redis命令可以参考`org.redisson.command.CommandAsyncService#sendCommand`方法,以及它的writeFuture complete后的回掉`org.redisson.command.CommandAsyncService#checkWriteFuture`方法
>
> 3) 发送成功netty等待redis响应的代码参考`org.redisson.client.handler.CommandDecoder#completeResponse`方法
>
> 4) retry则可以根据`org.redisson.command.CommandAsyncService#async`方法的引用,找到再各个超时和异常情况下重新调用async()且 `attempt` 参数+1来确认





04

—

### 阶段3: 量化指标



根据阶段2中可能的慢的部分,开发javaagent,使用asm库对各个位置进行插桩,采集指标.将指标通过prometheus暴露到grafana上观测,具体指标如下:

> 1) `retry`,表示一条redis指令正在执行第几次重试
>
> 2) `timeout`,表示一条redis指令发送成功到收到响应的耗时
>
> 3) `waitQueue`,表示正在执行的数量超过最大连接限制的时候,排队等待获取连接的指令数量
>
> 4) 连接池中各种`freeConnections`,`allConnnections`,`isFreezed`,等等等







05

—

### 阶段4: 混沌工程,压测,结合指标,发现问题



重新进行压测,使用chaosblade添加网络丢包,延迟,并且观察grafana上的指标,来确定问题。

![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6GZpkic3nvFBRz5moCBC2F0HXtTAtDp1n8JXXLiazcS0SGx1yDClkb1tw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 上图为增加指标后,使用使用chaosblade对redis所在服务端口执行50%丢包,200ms延迟实验后的结果
>
> 实验开始与14:26分附近,结束与14:31分附近,由于是分开执行的,所以服务器之间的实验有半分钟左右的执行时差

通过指标,我们发现如下现象:

在出现网络抖动以后,retry指标并没有完全停止增长!而是持续增高,waitQueue恢复为0, timeout也完全不增长,到此,确定应该要从为什么会网络正常后还触发retry进行定位具体问题.

开始使用arthas进行详细观察定位,详细定位流程如下:

1) 使用`monitor`命令,观测 `attempt` > 0的`async()`方法调用点(attempt为retry次数),观察请求参数是否有什么特殊,但是未能发现异常.使用arthas热更新,修改 默认的`RoundRobinLoadBalancer.getEntry()`发现两个slave连接池均存在有问题连接.

> 备注1: arthas热更新方案可以参考[这里](http://hengyunabc.github.io/arthas-online-hotswap/)
>
> 备注2: 原本还计划使用monitor命令进行retry的统计分析,但是由于monitor命令不支持condition-express,无法达到目的,后续给arthas提交了pr,给monitor添加condition-express参数,已合并至3.4.0详见[这里](https://github.com/alibaba/arthas/pull/1420)



2) 使用`stack`命令,观察 `attempt` > 0的`async()`方法是从哪里被调用的,结果发现网络恢复后的所有调用均来自于`org.redisson.command.CommandAsyncService$6`的`run()`方法.

![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic69YjsC6EwbbogPp3otoMKHicneWdgCER5ZAELwONooOWPSYHeauuiaUeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



3) 由于`org.redisson.command.CommandAsyncService$6`是一个匿名类,使用jad命令反编译,确定这是一个TimeTask,并且是用于检测获取连接超时以及发送指令成功的共用超时检测器. 这里根据代码确定是步骤1,2共用的超时器,基本上问题可以锁定到1,2之间了.



![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6eSpQ9MbiaLwbEnn1ACNic5jLz6mMbDMDbIUj4U4SDNPfW0pwicRhUmhbA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



4) 由于`retry`执行`async()`的时候是不携带失败连接的,所以重新使用monitor观察TimerTask的run方法,定位超时的时候到底取到连接了没?以及取到的连接的具体信息(details的connectionFuture),结果发现触发超时的时候,所有的 `ConnectionFuture` 都是成功获取到连接对象的,那么只有可能是发送redis指令到netty检测发送成功的这个过程超时了.同时指标图像在网络恢复后,waitQueue很快趋近与0,也很好的佐证了这个设想.



![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6ecA01FT9jx0FgLrER5gEt9HYL5icnclqw48Rkgko1gBicueV0qX1jgtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5) 使用更具体的观察表达式,print出了出问题的连接的channel详情,获取了其中`NioSocketChannel`中的状态,并且print出了localAddress,localPort,remoteAddress,remotePort,使用netstat -nt | grep localPort,最终发现,这个连接对象无论是channel状态还是os层的tcp连接状态,都已经确实关闭了.

**阶段结论** : 有某些已经关闭的连接始终在freeConnectionns中,即使超时后,依旧会被releaseConnection放回freeConnections,导致连接池中始终有一部分空闲连接实际上是关闭的,取到这个连接对象的业务线程只能触发重试机制.



06

—

### 阶段5: 详细定位问题,和官方沟通

在redisson项目issue中提交现象,但是官方表示使用 `ConnectionWatchdog` `PingConnectionHandler` 组合进行 `RedisConnection`内部channel的重连,对出现的问题表示不解,至此,陷入僵局.

重新对 `RedisConnection`的代码进行阅读的过程中,发现 `RedisConnection`中有一个 `closed`字段用来标示是否外部主动调用关闭,与channel的状态是相互隔离的,这个状态只能通过 `closeAsync()`方法修改,并且执行这个方法后,内部channel一定会被关闭(或者将被关闭),并且 `ConnectionWatchdog`并不会对主动关闭的 `RedisConnection`对象进行重连尝试.

ConnectionWatchDog channelInactive事件

![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6AfIyAJNKBhWCKavk9E1oKgwRNu0wzvcRTiaHJiaWzXqZfEKS2RYpyeWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)RedisConnection CloseAsync方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6laVxDEoSc6Oicr9siaHeiacRKl4ic2d8WVfibT0icZibUdUzYZgSr5hkAqjicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

RedisConnection close方法

![图片](https://mmbiz.qpic.cn/mmbiz_png/yiaMkmW2kMZHaAg7VyZuMBVtXTGP056ic6ibv0ShdTk16zwXuev7Sz1eRZxX8GDDbScUWy7HpbriajC3plae0SQ3lQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

进一步怀疑,是否在某种情况下, `RedisConnection`对象被检测关闭,并且从 `allConnections`中关闭,并移除,但是由于这个对象正在被使用,所以不在 `freeConnections`中,未能从 `freeConnections`中移除,之后在执行结束或者超时后,被重新 `add`回了 `freeConnection`(这里显然有个bug,虽然add回freeconnection的时候判断了connections的freezed,但是并没有判断连接对象是否已经主动关闭或者当前连接对象所属的代数),结合触发up,down次数多的时候,出现 `freeConnections.size()`> `allConnections.size()`,感觉很可能就是闪断引发的这个问题.

最后,我们打了一个heapdump,直接观察freeConnections中的RedisConnection的状态,确实发现了很多closed为true的连接对象,坐实了问题.

> 1) 测试图像中的freezed指标能看出,在弱网络情况下,连接池对象是会被被senitinel判断为下线,之后马上又重新判定为上线的,这个过程我描述为闪断
>
> 2) sentinel判定下线和上限没有确定是主观还是客观,另外从sentinel获取节点状态,默认是以1s的频率从sentinel server 拉取的,所以闪断过程可能在1s内发生



07

—

### 阶段6: 修复问题

根据问题,在 `releaseConnection()`的时候,加入对`RedisConnection`的`closed`状态的判断,如果是主动关闭连接,不加回`freeConnections`中,之后在本地重新验证,解决了这个bug,并提交PR到官方master分支,列入3.13.3里程碑.

issue: https://github.com/redisson/redisson/issues/2947

PR: https://github.com/redisson/redisson/pull/2956