# Arthas排查Kubernetes中的应用频繁挂掉重启问题



# 前言

其实最终定位到的问题还是蛮好解决的，但是因为应用在Kubernetes容器中的特殊性,导致在使用Arthas过程中出现了各种问题，所以单独成文和大家分享下。照例先讲下问题发生的背景，一个很老的web系统部署在tomcat容器里。近期打成了镜像丢到了Kubernetes环境中运行，总是各种挂，在Kubernetes层面定位了很久没找到具体问题，但是初步定位到是因为系统中的报表导出接口导致的问题，最后使用Arthas找到问题并解决。

# Kubernetes容器的特殊性

首先说下，我们的Kubernetes容器中运行的应用都是基于自己构建的基础镜像打包的，如JDK，和tomcat基础镜像，为了减小打包后应用的体积，我们对JDK进行了大量的删减，只保留了最小的jre运行时环境。这样导致的后果是在应用出现问题需要使用jdk工具时，如jinfo、jmap、jstack等都没了，没了这些工具就相当于一个战士没了武器，束手无策了，但是最后忽然想到了Arthas，java线上排错利器。

# 使用到的工具Arthas

Arthas是阿里巴巴开源的一款在线诊断java应用程序的工具，是greys工具的升级版本，深受开发者喜爱。当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？
7. Arthas采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。

项目地址：https://github.com/alibaba/arthas

# Arthas的使用

第一步：下载arthas-boot.jar

```
wget https://alibaba.github.io/arthas/arthas-boot.jar
```

第二步：运行

```
java -jar arthas-boot.jar
```

运行后，并没有出现熟悉的java进程选择界面，而是一闪而过，如下：

```
/opt/kl # java -jar arthas-boot.jar
[INFO] arthas-boot version: 3.1.0
[INFO] Can not find java process. Try to pass <pid> in command line.
Please select an available pid.
```

按官方的说明文档描述，假如出现了找不到pid的情况，在当前目录下应该会输出相关的log，但是我看了并没有日志，那只能尝试是否可以手动指定pid来使用Arthas了。在官网没找到类似说明，只能试试java -jar arthas-boot.jar -help看下，输出如下

```
/opt/kl # java -jar arthas-boot.jar -help
[INFO] arthas-boot version: 3.1.0
Usage: arthas-boot [-h] [--target-ip <value>] [--telnet-port <value>]
       [--http-port <value>] [--session-timeout <value>] [--arthas-home <value>]
       [--use-version <value>] [--repo-mirror <value>] [--versions] [--use-http]
       [--attach-only] [-c <value>] [-f <value>] [--height <value>] [--width
       <value>] [-v] [pid]

Bootstrap Arthas

EXAMPLES:
  java -jar arthas-boot.jar <pid>
  java -jar arthas-boot.jar --target-ip 0.0.0.0
  java -jar arthas-boot.jar --telnet-port 9999 --http-port -1
  java -jar arthas-boot.jar -c 'sysprop; thread' <pid>
  java -jar arthas-boot.jar -f batch.as <pid>
  java -jar arthas-boot.jar --use-version 3.0.5
  java -jar arthas-boot.jar --versions
  java -jar arthas-boot.jar --session-timeout 3600
  java -jar arthas-boot.jar --attach-only
  java -jar arthas-boot.jar --repo-mirror aliyun --use-http
```

EXAMPLES的第一条显示出来了，直接在jar后面加上pid即可，执行后，还是不行，输出如下：

```
/opt/kl # java -jar arthas-boot.jar 1
[INFO] arthas-boot version: 3.1.0
[INFO] Start download arthas from remote server: https://maven.aliyun.com/repository/public/com/taobao/arthas/arthas-packaging/3.1.0/arthas-packaging-3.1.0-bin.zip
[INFO] Download arthas success.
[INFO] arthas home: /root/.arthas/lib/3.1.0/arthas
[INFO] Try to attach process 1
Exception in thread "main" java.lang.IllegalArgumentException: Can not find tools.jar under java home: /usr/java/jdk/jre1.8.0_191, please try to start arthas-boot with full path java. Such as /opt/jdk/bin/java -jar arthas-boot.jar
        at com.taobao.arthas.boot.ProcessUtils.findJavaHome(ProcessUtils.java:195)
        at com.taobao.arthas.boot.ProcessUtils.startArthasCore(ProcessUtils.java:206)
        at com.taobao.arthas.boot.Bootstrap.main(Bootstrap.java:441)
```

### 异常解析：

根据异常可以明显看到说找不到tools.jar这个工具包了，还是回归到Kubernetes容器环境中因为精简了jre运行时环境导致jdk很多功能受限了。后面我做了一个非常规的事情，就是在完整的jdk中找到了这个tools.jar，丢到了jre里的lib目录中，继续尝试，但是还有问题，如下：

```
/opt/kl # java -jar arthas-boot.jar 1
[INFO] arthas-boot version: 3.1.0
[INFO] arthas home: /root/.arthas/lib/3.1.0/arthas
[INFO] Try to attach process 1
[ERROR] Start arthas failed, exception stack trace:
java.lang.UnsatisfiedLinkError: no attach in java.library.path
        at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1867)
        at java.lang.Runtime.loadLibrary0(Runtime.java:870)
        at java.lang.System.loadLibrary(System.java:1122)
        at sun.tools.attach.LinuxVirtualMachine.<clinit>(LinuxVirtualMachine.java:342)
        at sun.tools.attach.LinuxAttachProvider.attachVirtualMachine(LinuxAttachProvider.java:78)
        at com.sun.tools.attach.VirtualMachine.attach(VirtualMachine.java:250)
        at com.taobao.arthas.core.Arthas.attachAgent(Arthas.java:73)
        at com.taobao.arthas.core.Arthas.<init>(Arthas.java:26)
        at com.taobao.arthas.core.Arthas.main(Arthas.java:100)
[ERROR] attach fail, targetPid: 1
```

### 异常解析：

可以看到在补全了tools.jar之后，出现了新的异常信息java.lang.UnsatisfiedLinkError: no attach in java.library.path，表明可能我们缺的东西不是一点半点，我们知道attach功能是Arthas实现原理的两大原理之一。

attach：jdk1.6新增功能，通过attach机制，可以在jvm运行中，通过pid关联应用

instrument：jdk1.5新增功能，通过instrument俗称javaagent技术，可以修改jvm加载的字节码

然后Arthas和其他诊断工具一样，都是先通过attach链接上目标应用，通过instrument动态修改应用程序的字节码达到不重启应用而监控应用的目的

# 最后的救命稻草

使用完整的JDK中的java命令。

以上方式都试过不行之后，最后我把完整的JDK给下载到本地了，然后通过jdk的bin目录下的java命令启动arthas-boot.jar终于ok了，出现了熟悉的java进程选择界面：

```
/opt/kl/jdk1.8.0_191/bin # ./java -jar arthas-boot.jar
[INFO] arthas-boot version: 3.1.0
[INFO] Found existing java process, please choose one and hit RETURN.
* [1]: 1 org.apache.catalina.startup.Bootstrap
```

最后定位到的问题其实很简单，我记录了Arthas大盘中关于内存部分的图，如下：

[![img](https://camo.githubusercontent.com/01dc9cc25384763465b88d23785752eea55ca97ba21b21f79b3e74c79265973f/68747470733a2f2f6f7363696d672e6f736368696e612e6e65742f6f73636e65742f61373635633638386536356632316463646232373035343230373334623530336366662e6a7067)](https://camo.githubusercontent.com/01dc9cc25384763465b88d23785752eea55ca97ba21b21f79b3e74c79265973f/68747470733a2f2f6f7363696d672e6f736368696e612e6e65742f6f73636e65742f61373635633638386536356632316463646232373035343230373334623530336366662e6a7067)

上图从标题栏开始往下，分别是heap（堆内存）、eden_space（伊甸园区内存），survivor_space（幸存者区内存）、tenured_gen（老年代内存）。这张图是触发导出操作后的内存分布，堆内存从一开始的200M左右占用、到400M、到600M、一瞬间就飚升到900多兆了。最后从堆内存指标我们看到，总共989M，使用的内存已经飚升到988M了，这个时候其实应用已经挂了，Kubernetes容器已经在重启这个实例了。到这里基本已经到位到应用在容器中频繁挂掉重启问题的本质了。

### 但是为什么堆内存会这么小呢？

最终查明，有方面的原因：

1、因为我们这边都是spring boot应用，只有一个遗留的tomcat部署的应用，所以在镜像优化方面更偏向jdk基础基础镜像，而tomcat镜像没怎么关注，一开始对堆内存这块并没调优设置。

2、后面出现问题后，也确实想到过因为是导出报表导致应用挂掉，很可能是内存问题，设置过tomcat镜像内的堆内存大小，但是因为我们重新打包的镜像没有使用新的版本号，而是直接覆盖之前的版本，又使用Jenkins构建的，Jenkins所在主机拉过之前的镜像，导致镜像更新后，Jenkins打包时并没有去拉最新的调优过基础镜像。

# 解决问题

1、调优过的镜像加上新的版本号，让应用基于新的版本号构建镜像。或者清理下Jenkins所在主机的镜像，这个会导致第一次构建时速度变慢

2、优化导出报表的实现，我给的方案是，在导出大数据报表时，可以通过id分区，分别作业写入同一个服务器本地的文件中，然后让web容器映射下这个文件所在的目录，等所有分区的任务都结束后，直接组装一个文件下载链接返回给前端，让前端触发一次读文件操作即可。

# 最后尝试下jmap

除了使用Arthas外，最后还尝试了使用jmap工具，但是因为重新下载的JDK版本和主机jre版本不兼容，所有没用上。最后通过JDK发行的归档页面找到了对应的版本，还是成功的使用jmap -heap pid看到了内存情况,。内存分布也蛮清晰的，如：

```
/opt/kl/jdk1.8.0_191/bin # ./jmap -heap 1
Attaching to process ID 1, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.191-b12

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 3221225472 (3072.0MB)
   NewSize                  = 1073741824 (1024.0MB)
   MaxNewSize               = 1073741824 (1024.0MB)
   OldSize                  = 2147483648 (2048.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 966393856 (921.625MB)
   used     = 856023096 (816.3672409057617MB)
   free     = 110370760 (105.25775909423828MB)
   88.57911199302988% used
Eden Space:
   capacity = 859045888 (819.25MB)
   used     = 787314712 (750.8418197631836MB)
   free     = 71731176 (68.4081802368164MB)
   91.6499017104893% used
From Space:
   capacity = 107347968 (102.375MB)
   used     = 68708384 (65.52542114257812MB)
   free     = 38639584 (36.849578857421875MB)
   64.00529537736568% used
To Space:
   capacity = 107347968 (102.375MB)
   used     = 0 (0.0MB)
   free     = 107347968 (102.375MB)
   0.0% used
tenured generation:
   capacity = 2147483648 (2048.0MB)
   used     = 1521987528 (1451.4804153442383MB)
   free     = 625496120 (596.5195846557617MB)
   70.87306715548038% used
```

jdk归档下载页：[https://www.oracle.com/technetwork/java/javase/downloads](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html)

# 结语

Arthas是一个非常不错的工具，我们线上多次使用Arthas定位过问题。不过像今天的这种Kubernetes容器环境的话，因为本身运行时环境的缺失，可能使用的时候会存在各种问题，这里主要还是分享个思路。希望能给类似场景的同学提供一个有用的参考。