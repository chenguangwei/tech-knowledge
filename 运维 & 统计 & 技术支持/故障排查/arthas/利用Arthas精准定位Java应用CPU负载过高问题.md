# 利用Arthas精准定位Java应用CPU负载过高问题

最近我们线上有个应用服务器有点上头，CPU总能跑到99%，我寻思着它流量也不大啊，为啥能把自己整这么累？于是我登上这台服务器，看看它到底在干啥！

以前碰到类似问题，可能会考虑使用`top -Hp` 加 `jstack`命令去排查，虽然能大致定位到问题范围，但有效信息还是太少了，多数时候还是要靠猜。

今天向大家推荐一款更高效更精准的工具：`Arthas`！

Arthas 是Alibaba开源的Java诊断工具，能够帮助我们快速定位线上问题。基本的安装使用可以参考官方文档：https://alibaba.github.io/arthas
这次我们利用它来排查CPU负载高的问题。

CPU负载过高一般是某个或某几个线程有问题，所以我们尝试使用第一个命令：`thread`，这个命令会显示所有线程的信息，并且把CPU使用率高的线程排在前面。

```
[arthas@384]$ thread
Threads Total: 112, NEW: 0, RUNNABLE: 26, BLOCKED: 0, WAITING: 31, TIMED_WAITING: 55, TERMINATED: 0
ID  NAME    STATE           %CPU    TIME
108 h..ec-0 RUNNABLE        51      4011:48     
100 h..ec-2 RUNNABLE        48      4011:51
56  Ac..n   TIMED_WAITING   0       0:0
...
```

> 为了方便阅读，删掉了一些不重要的信息

可以看到，CPU资源几乎被前两个线程占满，并且已经执行了4000多分钟，我们服务器也就启动了两天，可见这两天它们是一刻也没闲着！

那它们究竟在干什么呢？我们可以使用命令：`thread id`，查看线程堆栈

```
[arthas@384]$ thread 108
"http-nio-7001-exec-10" Id=108 cpuUsage=51% RUNNABLE
    at c.g.c.c.HashBiMap.seekByKey(HashBiMap.java)
    at c.g.c.c.HashBiMap.put(HashBiMap.java:270)
    at c.g.c.c.HashBiMap.forcePut(HashBiMap.java:263)
    at c.y.r.j.o.OaInfoManager.syncUserCache(OaInfoManager.java:159)
```

> 也可以使用thread -n 3命令打印出CPU占比最高的前三个线程，这差不多是`top -Hp` & `printf` & `jstack` 三令合一的效果了

可以看到，这个线程一直在执行`HashBiMap.seekByKey`方法（可以重复执行几次`thread id`确保该线程执行的方法没有时刻在变化），造成这个问题一般有两个原因：

1. `seekByKey`方法被循环调用
2. `seekByKey`内部有死循环

先看一下是不是第一种，我们使用tt命令监听一下这个方法的调用情况

```
tt -t com.google.common.collect.HashBiMap seekByKey -n 100
```

**注意：在线上执行这个命令的时候，一定要记得加上 -n 参数，否则线上巨大的流量可能会瞬间撑爆你的JVM内存**

执行结果显示，`seekByKey`方法并没有被一直调用，那大概率是`seekByKey`方法内部有死循环。
看下这个方法内部的逻辑，我们可以使用`jad com.google.common.collect.HashBiMap seekByKey`命令反编译这个方法，这样做的好处是显得比较高端，不过我还是打算直接找到源码，说不定还有注释。

源码如下：

```
 private BiEntry<K, V> seekByKey(@Nullable Object key, int keyHash) {
    for (BiEntry<K, V> entry = hashTableKToV[keyHash & mask];
        entry != null;
        entry = entry.nextInKToVBucket) {
      if (keyHash == entry.keyHash && Objects.equal(key, entry.key)) {
        return entry;
      }
    }
    return null;
  }
```

然后并没有注释，还好这个方法逻辑比较简单，也很容易看懂。

1. 通过hash找到bucket，每个bucket是一个链表
2. 遍历链表，找到这个key对应的entry。这里要留意下entry的下一个节点是**nextInKToVBucket**，后文中会用到

发生了死循环，我们猜想可能是因为这个链表有环路。那么有没有办法验证这个猜想呢？

答案是**有**！那么如何验证呢？

首先我们要获得这个`HashBiMap`对象，以便于查询对象里的数据。获得这个对象有很多办法，比如监听这个对象的某个方法，然后主动触发这个方法。这里向大家介绍一种更为通用的方法，这个方法在SpringMVC程序里非常好用。

因为我们是SpringMVC应用，所有请求都会被`RequestMappingHandlerAdapter`拦截，我们通过tt命令，监听`invokeHandlerMethod`的执行，然后在页面随便点点，就会得到以下内容：

```
[arthas@384]$ tt -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod -n 10
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 622 ms.

 INDEX	COST(ms)  	OBJECT    	CLASS             METHOD                                             
------------------------------------------------------------------------------------
 1000 	481.203383	0x481eb705	RequestMappingHandlerAdapter    invokeHandlerMethod                                
 1001 	3.432024  	0x481eb705	RequestMappingHandlerAdapter	invokeHandlerMethod                               
...
```

tt 命令会记录方法调用时的所有入参和返回值、抛出的异常、对象本身等数据。INDEX字段代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作。

我们可以通过 -i 参数后边跟着对应的 INDEX 编号查看这条记录的详细信息。再通过-w参数，指定一个OGNL表达式，查找相关对象

```
[arthas@384]$ tt -i 1000 -w 'target.getApplicationContext()'
@AnnotationConfigServletWebServerApplicationContext[
    reader=@AnnotatedBeanDefinitionReader[org.springframework.context.annotation.AnnotatedBeanDefinitionReader@50294e97],
    scanner=@ClassPathBeanDefinitionScanner[org.springframework.context.annotation.ClassPathBeanDefinitionScanner@5eeeaae2],
    annotatedClasses=@LinkedHashSet[isEmpty=true;size=0],
    basePackages=null,
```

> OGNL使用文档：https://commons.apache.org/proper/commons-ognl/language-guide.html

Arthas会把当前执行的对象放到target变量中，通过`target.getApplicationContext()`就得到了`SpringContext`对象，然后，我们就可以**为所欲为**了!

接下来我们需要用OGNL写一个函数，来实现链表的环路检测，在OGNL里写一段环路检测代码里是不太容易的，这里我用了一个取巧的伪实现（有更好思路的欢迎在评论区留言）

```
#loopCnt=0, 
#foundCycle=:[ #this == null ? false :  
    #loopCnt > 50 ? true : 
        (
            #loopCnt = #loopCnt + 1, 
            #foundCycle(#this.nextInKToVBucket)
        )
```

因为我知道一个bucket不太可能有50个以上的节点，所以就通过遍历次数是否大于50来判断是否有环路。

完整的命令：

> tt -i 1000 -w 'target.getApplicationContext().getBean("oaInfoManager").userCache.entrySet().{delegate}.{^ #loopCnt = 0,#foundCycle = :[ #this == null ? false : #loopCnt > 50 ? true : (#loopCnt = #loopCnt + 1, #foundCycle(#this.nextInKToVBucket))], #foundCycle(#this)}.get(0)' -x 2

命令解析：

1. 获取`HashBiMap`对象：`target.getApplicationContext().getBean("oaInfoManager").userCache`
2. 遍历所有entry，取出第一个有环路的entry
3. -x 参数指定展开层级，我们需要将这个参数设置的比环要大一些，才能确保可以发现环路。这里我们的环路非常小，所以设置成了2

执行结果如下：

```
@BiEntry[
    key=@String[张三],
    value=@Long[1111],
    nextInKToVBucket=@BiEntry[
        key=@String[李四],
        value=@Long[2222],
        nextInKToVBucket=@BiEntry[张三=1111]
    ]
]
```

可以看到是有 张三->李四->张三 这样一个环路。至此，造成死循环的原因确定了下来。结合两个线程几乎同时启动，又同时在执行`HashBiMap.forcePut`方法，容易想到是因为**并发**导致了数据的不一致，这一点也可以验证，不过由于篇幅有限，这里就不再赘述。

找到了问题，就成功了99%，解决这个问题的方法非常简单，就是对`syncUserCache`方法加一个**synchronized**关键字！

#### 结语

这次遇到的问题并不复杂，用`jstack`命令也可以解决的了。但我们希望通过这样一个案例，向大家展示Arthas一些强大的功能，帮助大家打开思路，未来在遇到更复杂场景时，可以多一些趁手的工具！