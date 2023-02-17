# Arthas - Java 线上问题定位处理的终极利器





在使用 **Arthas** 之前，当遇到 Java 线上问题时，如 CPU 飙升、负载突高、内存溢出等问题，你需要查命令，查网络，然后 jps、jstack、jmap、jhat、jstat、hprof 等一通操作。最终焦头烂额，还不一定能查出问题所在。而现在，大多数的常见问题你都可以使用 **Arthas** 轻松定位，迅速解决，及时止损，准时下班。

# [#](https://www.wdbyte.com/2019/11/arthas/#_1%E3%80%81arthas-%E4%BB%8B%E7%BB%8D)1、Arthas 介绍

**Arthas** 是 `Alibaba` 在 2018 年 9 月开源的 **Java 诊断**工具。支持 `JDK6+`， 采用命令行交互模式，提供 `Tab` 自动补全，可以方便的定位和诊断线上程序运行问题。截至本篇文章编写时，已经收获 `Star` 17000+。

**Arthas** 官方文档十分详细，本文也参考了官方文档内容，同时在开源在的 `Github` 的项目里的 `Issues` 里不仅有问题反馈，更有大量的使用案例，也可以进行学习参考。

开源地址：*https://github.com/alibaba/arthas*

官方文档：*https://alibaba.github.io/arthas*

# [#](https://www.wdbyte.com/2019/11/arthas/#_2%E3%80%81arthas-%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)2、Arthas 使用场景

得益于 **Arthas** 强大且丰富的功能，让 **Arthas** 能做的事情超乎想象。下面仅仅列举几项常见的使用情况，更多的使用场景可以在熟悉了 **Arthas** 之后自行探索。

1. 是否有一个全局视角来查看系统的运行状况？
2. 为什么 CPU 又升高了，到底是哪里占用了 CPU ？
3. 运行的多线程有死锁吗？有阻塞吗？
4. 程序运行耗时很长，是哪里耗时比较长呢？如何监测呢？
5. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
6. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
7. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
8. 有什么办法可以监控到 JVM 的实时运行状态？

# [#](https://www.wdbyte.com/2019/11/arthas/#_3%E3%80%81arthas-%E6%80%8E%E4%B9%88%E7%94%A8)3、Arthas 怎么用

前文已经提到，**Arthas** 是一款命令行交互模式的 Java 诊断工具，由于是 Java 编写，所以可以直接下载相应 的 jar 包运行。

## [#](https://www.wdbyte.com/2019/11/arthas/#_3-1-%E5%AE%89%E8%A3%85)3.1 安装

可以在官方 Github 上进行下载，如果速度较慢，可以尝试国内的码云 Gitee 下载。

```
# github下载
wget https://alibaba.github.io/arthas/arthas-boot.jar
# 或者 Gitee 下载
wget https://arthas.gitee.io/arthas-boot.jar
# 打印帮助信息
java -jar arthas-boot.jar -h
```

## [#](https://www.wdbyte.com/2019/11/arthas/#_3-2-%E8%BF%90%E8%A1%8C)3.2 运行

**Arthas** 只是一个 java 程序，所以可以直接用 `java -jar` 运行。运行时或者运行之后要选择要监测的 Java 进程。

```
# 运行方式1，先运行，在选择 Java 进程 PID
java -jar arthas-boot.jar
# 选择进程(输入[]内编号(不是PID)回车)
[INFO] arthas-boot version: 3.1.4
[INFO] Found existing java process, please choose one and hit RETURN.
* [1]: 11616 com.Arthas
  [2]: 8676
  [3]: 16200 org.jetbrains.jps.cmdline.Launcher
  [4]: 21032 org.jetbrains.idea.maven.server.RemoteMavenServer

# 运行方式2，运行时选择 Java 进程 PID
java -jar arthas-boot.jar [PID]
```

查看 PID 的方式可以通过 `ps` 命令，也可以通过 JDK 提供的 `jps` 命令。

```
# 查看运行的 java 进程信息
$ jps -mlvV 
# 筛选 java 进程信息
$ jps -mlvV | grep [xxx] 
```

`jps` 筛选想要的进程方式。

![jps 筛选进程](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1570979767404.png)

在出现 **Arthas** Logo 之后就可以使用命令进行问题诊断了。下面会详细介绍。

![Arthas 启动](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/image-20191106003512451.png)

更多的启动方式可以参考 help 帮助命令。

```
# 其他用法
EXAMPLES:
  java -jar arthas-boot.jar <pid>
  java -jar arthas-boot.jar --target-ip 0.0.0.0
  java -jar arthas-boot.jar --telnet-port 9999 --http-port -1
  java -jar arthas-boot.jar --tunnel-server 'ws://192.168.10.11:7777/ws'
  java -jar arthas-boot.jar --tunnel-server 'ws://192.168.10.11:7777/ws'
--agent-id bvDOe8XbTM2pQWjF4cfw
  java -jar arthas-boot.jar --stat-url 'http://192.168.10.11:8080/api/stat'
  java -jar arthas-boot.jar -c 'sysprop; thread' <pid>
  java -jar arthas-boot.jar -f batch.as <pid>
  java -jar arthas-boot.jar --use-version 3.1.4
  java -jar arthas-boot.jar --versions
  java -jar arthas-boot.jar --session-timeout 3600
  java -jar arthas-boot.jar --attach-only
  java -jar arthas-boot.jar --repo-mirror aliyun --use-http 

```

## [#](https://www.wdbyte.com/2019/11/arthas/#_3-3-web-console)3.3 web console

**Arthas** 目前支持 `Web Console`，在成功启动连接进程之后就已经自动启动，可以直接访问 http://127.0.0.1:8563/ 访问，页面上的操作模式和控制台完全一样。

![1570979937637](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1570979937637.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_3-4-%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)3.4 常用命令

下面列举一些 [**Arthas** (opens new window)](https://www.wdbyte.com/2019/11/arthas/)的常用命令，看到这里你可能还不知道怎么使用，别急，后面会一一介绍。

| 命令                                                                                 | 介绍                                                |
| ---------------------------------------------------------------------------------- | ------------------------------------------------- |
| [dashboard(opens new window)](https://alibaba.github.io/arthas/dashboard.html)     | 当前系统的实时数据面板                                       |
| [**thread**(opens new window)](https://alibaba.github.io/arthas/thread.html)       | 查看当前 JVM 的线程堆栈信息                                  |
| [**watch**(opens new window)](https://alibaba.github.io/arthas/watch.html)         | 方法执行数据观测                                          |
| **[trace(opens new window)](https://alibaba.github.io/arthas/trace.html)**         | 方法内部调用路径，并输出方法路径上的每个节点上耗时                         |
| [**stack**(opens new window)](https://alibaba.github.io/arthas/stack.html)         | 输出当前方法被调用的调用路径                                    |
| [**tt**(opens new window)](https://alibaba.github.io/arthas/tt.html)               | 方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测 |
| [monitor(opens new window)](https://alibaba.github.io/arthas/monitor.html)         | 方法执行监控                                            |
| [jvm(opens new window)](https://alibaba.github.io/arthas/jvm.html)                 | 查看当前 JVM 信息                                       |
| [vmoption(opens new window)](https://alibaba.github.io/arthas/vmoption.html)       | 查看，更新 JVM 诊断相关的参数                                 |
| [sc(opens new window)](https://alibaba.github.io/arthas/sc.html)                   | 查看 JVM 已加载的类信息                                    |
| [sm(opens new window)](https://alibaba.github.io/arthas/sm.html)                   | 查看已加载类的方法信息                                       |
| [jad(opens new window)](https://alibaba.github.io/arthas/jad.html)                 | 反编译指定已加载类的源码                                      |
| [classloader(opens new window)](https://alibaba.github.io/arthas/classloader.html) | 查看 classloader 的继承树，urls，类加载信息                    |
| [heapdump(opens new window)](https://alibaba.github.io/arthas/heapdump.html)       | 类似 jmap 命令的 heap dump 功能                          |

## [#](https://www.wdbyte.com/2019/11/arthas/#_3-5-%E9%80%80%E5%87%BA)3.5 退出

使用 shutdown 退出时 **Arthas** 同时自动重置所有增强过的类 。

# [#](https://www.wdbyte.com/2019/11/arthas/#_4%E3%80%81arthas-%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C)4、Arthas 常用操作

上面已经了解了什么是 **Arthas**，以及 **Arthas** 的启动方式，下面会依据一些情况，详细说一说 **Arthas** 的使用方式。在使用命令的过程中如果有问题，每个命令都可以是 `-h` 查看帮助信息。

首先编写一个有各种情况的测试类运行起来，再使用 **Arthas** 进行问题定位，

```
import java.util.HashSet;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import lombok.extern.slf4j.Slf4j;

/** * <p> * Arthas Demo * 公众号：程序猿阿朗 * * @Author niujinpeng */
@Slf4j
public class Arthas {

    private static HashSet hashSet = new HashSet();
    /** 线程池，大小1*/
    private static ExecutorService executorService = Executors.newFixedThreadPool(1);

    public static void main(String[] args) {
        // 模拟 CPU 过高，这里注释掉了，测试时可以打开
        // cpu();
        // 模拟线程阻塞
        thread();
        // 模拟线程死锁
        deadThread();
        // 不断的向 hashSet 集合增加数据
        addHashSetThread();
    }

    /**     * 不断的向 hashSet 集合添加数据     */
    public static void addHashSetThread() {
        // 初始化常量
        new Thread(() -> {
            int count = 0;
            while (true) {
                try {
                    hashSet.add("count" + count);
                    Thread.sleep(10000);
                    count++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    public static void cpu() {
        cpuHigh();
        cpuNormal();
    }

    /**     * 极度消耗CPU的线程     */
    private static void cpuHigh() {
        Thread thread = new Thread(() -> {
            while (true) {
                log.info("cpu start 100");
            }
        });
        // 添加到线程
        executorService.submit(thread);
    }

    /**     * 普通消耗CPU的线程     */
    private static void cpuNormal() {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                while (true) {
                    log.info("cpu start");
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    /**     * 模拟线程阻塞,向已经满了的线程池提交线程     */
    private static void thread() {
        Thread thread = new Thread(() -> {
            while (true) {
                log.debug("thread start");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        // 添加到线程
        executorService.submit(thread);
    }

    /**     * 死锁     */
    private static void deadThread() {
        /** 创建资源 */
        Object resourceA = new Object();
        Object resourceB = new Object();
        // 创建线程
        Thread threadA = new Thread(() -> {
            synchronized (resourceA) {
                log.info(Thread.currentThread() + " get ResourceA");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info(Thread.currentThread() + "waiting get resourceB");
                synchronized (resourceB) {
                    log.info(Thread.currentThread() + " get resourceB");
                }
            }
        });

        Thread threadB = new Thread(() -> {
            synchronized (resourceB) {
                log.info(Thread.currentThread() + " get ResourceB");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info(Thread.currentThread() + "waiting get resourceA");
                synchronized (resourceA) {
                    log.info(Thread.currentThread() + " get resourceA");
                }
            }
        });
        threadA.start();
        threadB.start();
    }
}
```

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-1-%E5%85%A8%E5%B1%80%E7%9B%91%E6%8E%A7)4.1 全局监控

使用 **dashboard** 命令可以概览程序的 线程、内存、GC、运行环境信息。

![dashboard](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571212470373.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-2-cpu-%E4%B8%BA%E4%BB%80%E4%B9%88%E8%B5%B7%E9%A3%9E%E4%BA%86)4.2 CPU 为什么起飞了

上面的代码例子有一个 `CPU` 空转的死循环，非常的消耗 `CPU性能`，那么怎么找出来呢？

使用 **thread** 查看 **所有**线程信息，同时会列出每个线程的 `CPU` 使用率，可以看到图里 ID 为 12 的线程 CPU 使用 100%。 ![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1570983440457.png)

使用命令 **thread 12** 查看 CPU 消耗较高的 12 号线程信息，可以看到 CPU 使用较高的方法和行数（这里的行数可能和上面代码里的行数有区别，因为上面的代码在我写文章时候重新排过版了）。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1570983401254.png)

上面是先通过观察总体的线程信息，然后查看具体的线程运行情况。如果只是为了寻找 CPU 使用较高的线程，可以直接使用命令 **thread -n [显示的线程个数]** ，就可以排列出 CPU 使用率 **Top N** 的线程。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1570983061047.png)

定位到的 CPU 使用最高的方法。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571016675083.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-3-%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81)4.3 线程池线程状态

定位线程问题之前，先回顾一下线程的几种常见状态：

- **RUNNABLE** 运行中
- **TIMED_WAITIN** 调用了以下方法的线程会进入 **TIMED_WAITING**：
  1. Thread#sleep()
  2. Object#wait () 并加了超时参数
  3. Thread#join () 并加了超时参数
  4. LockSupport#parkNanos()
  5. LockSupport#parkUntil()
- **WAITING** 当线程调用以下方法时会进入 WAITING 状态：
  1. Object#wait () 而且不加超时参数
  2. Thread#join () 而且不加超时参数
  3. LockSupport#park()
- **BLOCKED** 阻塞，等待锁

上面的模拟代码里，定义了线程池大小为 1 的线程池，然后在 `cpuHigh` 方法里提交了一个线程，在 `thread` 方法再次提交了一个线程，后面的这个线程因为线程池已满，会阻塞下来。

使用 **thread | grep pool** 命令查看线程池里线程信息。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571020871537.png)

可以看到线程池有 **WAITING** 的线程。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571021838323.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-4-%E7%BA%BF%E7%A8%8B%E6%AD%BB%E9%94%81)4.4 线程死锁

上面的模拟代码里 `deadThread` 方法实现了一个死锁，使用 **thread -b** 命令查看直接定位到死锁信息。

```
/** * 死锁 */
private static void deadThread() {
    /** 创建资源 */
    Object resourceA = new Object();
    Object resourceB = new Object();
    // 创建线程
    Thread threadA = new Thread(() -> {
        synchronized (resourceA) {
            log.info(Thread.currentThread() + " get ResourceA");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info(Thread.currentThread() + "waiting get resourceB");
            synchronized (resourceB) {
                log.info(Thread.currentThread() + " get resourceB");
            }
        }
    });

    Thread threadB = new Thread(() -> {
        synchronized (resourceB) {
            log.info(Thread.currentThread() + " get ResourceB");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info(Thread.currentThread() + "waiting get resourceA");
            synchronized (resourceA) {
                log.info(Thread.currentThread() + " get resourceA");
            }
        }
    });
    threadA.start();
    threadB.start();
} 

```

检查到的死锁信息。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571206638142.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-5-%E5%8F%8D%E7%BC%96%E8%AF%91)4.5 反编译

上面的代码放到了包 `com` 下，假设这是一个线程环境，当怀疑当前运行的代码不是自己想要的代码时，可以直接反编译出代码，也可以选择性的查看类的字段或方法信息。

如果怀疑不是自己的代码，可以使用 **jad** 命令直接反编译 class。

![jad](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/image-20191106012005747.png)

`jad` 命令还提供了一些其他参数：

```
# 反编译只显示源码
jad --source-only com.Arthas
# 反编译某个类的某个方法
jad --source-only com.Arthas mysql
```

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-6-%E6%9F%A5%E7%9C%8B%E5%AD%97%E6%AE%B5%E4%BF%A1%E6%81%AF)4.6 查看字段信息

使用 **sc -d -f ** 命令查看类的字段信息。

```
[arthas@20252]$ sc -d -f com.Arthas
sc -d -f com.Arthas
 class-info        com.Arthas
 code-source       /C:/Users/Niu/Desktop/arthas/target/classes/
 name              com.Arthas
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
 isLocalClass      false
 isMemberClass     false
 isPrimitive       false
 isSynthetic       false
 simple-name       Arthas
 modifier          public
 annotation
 interfaces
 super-class       +-java.lang.Object
 class-loader      +-sun.misc.Launcher$AppClassLoader@18b4aac2
                     +-sun.misc.Launcher$ExtClassLoader@2ef1e4fa
 classLoaderHash   18b4aac2
 fields            modifierfinal,private,static
                   type    org.slf4j.Logger
                   name    log
                   value   Logger[com.Arthas]

                   modifierprivate,static
                   type    java.util.HashSet
                   name    hashSet
                   value   [count1, count2]

                   modifierprivate,static
                   type    java.util.concurrent.ExecutorService
                   name    executorService
                   value   java.util.concurrent.ThreadPoolExecutor@71c03156[Ru
                           nning, pool size = 1, active threads = 1, queued ta
                           sks = 0, completed tasks = 0]


Affect(row-cnt:1) cost in 9 ms. 
```

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-7-%E6%9F%A5%E7%9C%8B%E6%96%B9%E6%B3%95%E4%BF%A1%E6%81%AF)4.7 查看方法信息

使用 **sm** 命令查看类的方法信息。

```
[arthas@22180]$ sm com.Arthas
com.Arthas <init>()V
com.Arthas start()V
com.Arthas thread()V
com.Arthas deadThread()V
com.Arthas lambda$cpuHigh$1()V
com.Arthas cpuHigh()V
com.Arthas lambda$thread$3()V
com.Arthas addHashSetThread()V
com.Arthas cpuNormal()V
com.Arthas cpu()V
com.Arthas lambda$addHashSetThread$0()V
com.Arthas lambda$deadThread$4(Ljava/lang/Object;Ljava/lang/Object;)V
com.Arthas lambda$deadThread$5(Ljava/lang/Object;Ljava/lang/Object;)V
com.Arthas lambda$cpuNormal$2()V
Affect(row-cnt:16) cost in 6 ms.  
```

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-8-%E5%AF%B9%E5%8F%98%E9%87%8F%E7%9A%84%E5%80%BC%E5%BE%88%E6%98%AF%E5%A5%BD%E5%A5%87)4.8 对变量的值很是好奇

使用 **ognl** 命令，ognl 表达式可以轻松操作想要的信息。

代码还是上面的示例代码，我们查看变量 `hashSet` 中的数据：

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571196786678.png)

查看静态变量 `hashSet` 信息。

```
[arthas@19856]$ ognl '@com.Arthas@hashSet'
@HashSet[
    @String[count1],
    @String[count2],
    @String[count29],
    @String[count28],
    @String[count0],
    @String[count27],
    @String[count5],
    @String[count26],
    @String[count6],
    @String[count25],
    @String[count3],
    @String[count24], 

```

查看静态变量 hashSet 大小。

```
[arthas@19856]$ ognl '@com.Arthas@hashSet.size()'
    @Integer[57]
```

1  
2  

甚至可以进行操作。

```
[arthas@19856]$ ognl  '@com.Arthas@hashSet.add("test")'
    @Boolean[true]
[arthas@19856]$
# 查看添加的字符
[arthas@19856]$ ognl  '@com.Arthas@hashSet' | grep test
    @String[test],
[arthas@19856]$ 
```

`ognl` 可以做很多事情，可以参考 [ognl 表达式特殊用法 (https://github.com/alibaba/arthas/issues/71) (opens new window)](https://github.com/alibaba/arthas/issues/71)。

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-9-%E7%A8%8B%E5%BA%8F%E6%9C%89%E6%B2%A1%E6%9C%89%E9%97%AE%E9%A2%98)4.9 程序有没有问题

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-9-1-%E8%BF%90%E8%A1%8C%E8%BE%83%E6%85%A2%E3%80%81%E8%80%97%E6%97%B6%E8%BE%83%E9%95%BF)4.9.1 运行较慢、耗时较长

使用 **trace** 命令可以跟踪统计方法耗时

这次换一个模拟代码。一个最基础的 Springboot 项目（当然，不想 Springboot 的话，你也可以直接在 UserController 里 main 方法启动）控制层 `getUser` 方法调用了 `userService.get(uid);`，这个方法中分别进行 `check`、`service`、`redis`、`mysql` 操作。

```
@RestController
@Slf4j
public class UserController {

    @Autowired
    private UserServiceImpl userService;

    @GetMapping(value = "/user")
    public HashMap<String, Object> getUser(Integer uid) throws Exception {
        // 模拟用户查询
        userService.get(uid);
        HashMap<String, Object> hashMap = new HashMap<>();
        hashMap.put("uid", uid);
        hashMap.put("name", "name" + uid);
        return hashMap;
    }
}  
```

模拟代码 Service:

```
@Service
@Slf4j
public class UserServiceImpl {

    public void get(Integer uid) throws Exception {
        check(uid);
        service(uid);
        redis(uid);
        mysql(uid);
    }

    public void service(Integer uid) throws Exception {
        int count = 0;
        for (int i = 0; i < 10; i++) {
            count += i;
        }
        log.info("service  end {}", count);
    }

    public void redis(Integer uid) throws Exception {
        int count = 0;
        for (int i = 0; i < 10000; i++) {
            count += i;
        }
        log.info("redis  end {}", count);
    }

    public void mysql(Integer uid) throws Exception {
        long count = 0;
        for (int i = 0; i < 10000000; i++) {
            count += i;
        }
        log.info("mysql end {}", count);
    }

      public boolean check(Integer uid) throws Exception {
         if (uid == null || uid < 0) {
             log.error("uid不正确，uid:{}", uid);
             throw new Exception("uid不正确");
         }
         return true;
     }
}
```



运行 Springboot 之后，使用 **trace== ** 命令开始检测耗时情况。

```
[arthas@6592]$ trace com.UserController getUser
```



访问接口 `/getUser` ，可以看到耗时信息，看到 `com.UserServiceImpl:get()` 方法耗时较高。 ![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571208153793.png)

继续跟踪耗时高的方法，然后再次访问。

```
[arthas@6592]$ trace com.UserServiceImpl get
```



![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571208245597.png)

很清楚的看到是 `com.UserServiceImpl` 的 `mysql` 方法耗时是最高的。

```
Affect(class-cnt:1 , method-cnt:1) cost in 31 ms.
`---ts=2019-10-16 14:40:10;thread_name=http-nio-8080-exec-8;id=1f;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@23a918c7
    `---[6.792201ms] com.UserServiceImpl:get()
        +---[0.008ms] com.UserServiceImpl:check() #17
        +---[0.076ms] com.UserServiceImpl:service() #18
        +---[0.1089ms] com.UserServiceImpl:redis() #19
        `---[6.528899ms] com.UserServiceImpl:mysql() #20
```



### [#](https://www.wdbyte.com/2019/11/arthas/#_4-9-2-%E7%BB%9F%E8%AE%A1%E6%96%B9%E6%B3%95%E8%80%97%E6%97%B6)4.9.2 统计方法耗时

使用 **monitor** 命令监控统计方法的执行情况。

每 5 秒统计一次 `com.UserServiceImpl` 类的 `get` 方法执行情况。

```
monitor -c 5 com.UserServiceImpl get
```

 

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571210158018.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-10-%E6%83%B3%E8%A7%82%E5%AF%9F%E6%96%B9%E6%B3%95%E4%BF%A1%E6%81%AF)4.10 想观察方法信息

下面的示例用到了文章的前两个模拟代码。

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-10-1-%E8%A7%82%E5%AF%9F%E6%96%B9%E6%B3%95%E7%9A%84%E5%85%A5%E5%8F%82%E5%87%BA%E5%8F%82%E4%BF%A1%E6%81%AF)4.10.1 观察方法的入参出参信息

使用 **watch** 命令轻松查看输入输出参数以及异常等信息。

```
 USAGE:
   watch [-b] [-e] [-x <value>] [-f] [-h] [-n <value>] [-E] [-M <value>] [-s] class-pattern method-pattern express [condition-express]

 SUMMARY:
   Display the input/output parameter, return object, and thrown exception of specified method invocation
   The express may be one of the following expression (evaluated dynamically):
           target : the object
            clazz : the object's class           method : the constructor or method           params : the parameters array of method     params[0..n] : the element of parameters array        returnObj : the returned object of method         throwExp : the throw exception of method         isReturn : the method ended by return          isThrow : the method ended by throwing exception            #cost : the execution time in ms of method invocation Examples:   watch -b org.apache.commons.lang.StringUtils isBlank params   watch -f org.apache.commons.lang.StringUtils isBlank returnObj   watch org.apache.commons.lang.StringUtils isBlank '{params, target, returnObj}' -x 2   watch -bf *StringUtils isBlank params   watch *StringUtils isBlank params[0]   watch *StringUtils isBlank params[0] params[0].length==1   watch *StringUtils isBlank params '#cost>100'
   watch -E -b org\.apache\.commons\.lang\.StringUtils isBlank params[0]

 WIKI:
   https://alibaba.github.io/arthas/watch  
```

常用操作：

```
# 查看入参和出参
$ watch com.Arthas addHashSet '{params[0],returnObj}'
# 查看入参和出参大小
$ watch com.Arthas addHashSet '{params[0],returnObj.size}'
# 查看入参和出参中是否包含 'count10'
$ watch com.Arthas addHashSet '{params[0],returnObj.contains("count10")}'
# 查看入参和出参，出参 toString
$ watch com.Arthas addHashSet '{params[0],returnObj.toString()}'
```



查看入参出参。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571196483469.png)

查看返回的异常信息。

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-10-2-%E8%A7%82%E5%AF%9F%E6%96%B9%E6%B3%95%E7%9A%84%E8%B0%83%E7%94%A8%E8%B7%AF%E5%BE%84)4.10.2 观察方法的调用路径

使用 **stack** 命令查看方法的调用信息。

```
# 观察 类com.UserServiceImpl的 mysql 方法调用路径
stack com.UserServiceImpl mysql
```



![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571210706602.png)

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-10-3-%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E6%97%B6%E7%A9%BA%E9%9A%A7%E9%81%93)4.10.3 方法调用时空隧道

使用 **tt** 命令记录方法执行的详细情况。

> **tt** 命令方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测 。

常用操作：

开始记录方法调用信息：tt -t com.UserServiceImpl check

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571212007249.png)

可以看到记录中 INDEX=1001 的记录的 IS-EXP = true ，说明这次调用出现异常。

查看记录的方法调用信息： tt -l

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571212080071.png)

查看调用记录的详细信息（-i 指定 INDEX）： tt -i 1001

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571212151064.png)

可以看到 INDEX=1001 的记录的异常信息。

重新发起调用，使用指定记录，使用 -p 重新调用。

```
tt -i 1001 -p
```

 

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2019/1571212227058.png)

## [#](https://www.wdbyte.com/2019/11/arthas/#_4-5-%E7%81%AB%E7%84%B0%E5%9B%BE%E5%88%86%E6%9E%90)4.5. 火焰图分析

最近 Arthas 性能分析工具上线了**火焰图**分析功能，Arthas 使用 **async-profiler** 生成 CPU / 内存火焰图进行性能分析，弥补了之前内存分析的不足。在 Arthas 上使用还是比较方便的。

**`profiler`** 命令支持生成应用热点的火焰图。本质上是通过不断的采样，然后把收集到的采样结果生成火焰图。

**`profiler`** 命令基本运行结构是 **`profiler action [actionArg]`**

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-5-1-%E4%BD%BF%E7%94%A8%E6%A1%88%E4%BE%8B)**4.5.1. 使用案例**

**开启 prifilter**

默认情况下，生成的是 cpu 的火焰图，即 event 为 `cpu`。可以用 `--event` 参数来指定，使用 start 命令开始捕获信息。

```
$ profiler start
Started [cpu] profiling
```

 

获取已采集的 sample 的数量

```
$ profiler getSamples
23
```



查看 profiler 状态，可以查看当前 profiler 在采样哪种 `event` 和进行的采样时间。

$ profiler status

[cpu] profiling is running for 4 seconds

**停止 profiler**

生成 svg 格式火焰图

```
$ profiler stop
profiler output file: /tmp/demo/arthas-output/20191125-135546.svg
OK
```



默认情况下，生成的结果保存到应用的`工作目录`下的 `arthas-output` 目录。可以通过 `--file` 参数来指定输出结果路径。

比如：

```
$ profiler stop --file /tmp/output.svg
```



**HTML 格式输出**

默认情况下，结果文件是 `svg` 格式，如果想生成 `html` 格式，可以用 `--format` 参数指定：$ profiler stop --format html

**查看 profilter**

默认情况下，arthas 使用 3658 端口，则可以打开： http://localhost:3658/arthas-output/ 查看到 `arthas-output` 目录下面的 profiler 结果：

![img](https://alibaba.github.io/arthas/_images/arthas-output.jpg)

点击可以查看具体的结果：**火焰图里，横条越长，代表使用的越多，从下到上是调用堆栈信息**

![img](https://alibaba.github.io/arthas/_images/arthas-output-svg.jpg)

**profilter 自持多种分析方式，** 常见的有 event: cpu|alloc|lock|cache-misses etc. 比如要分析内存使用情况。

$ profiler start --event alloc

### [#](https://www.wdbyte.com/2019/11/arthas/#_4-5-2-%E5%A4%8D%E6%9D%82%E5%91%BD%E4%BB%A4)4.5.2. 复杂命令

比如开始采样：

```
profiler execute 'start'
```



停止采样，并保存到指定文件里：

```
profiler execute 'stop,file=/tmp/result.svg'
```

1

文中代码已经上传到 [Github (opens new window)](https://github.com/niumoo/lab-notes/)。
