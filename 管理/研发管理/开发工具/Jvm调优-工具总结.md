# Jvm调优-工具总结

> 本文主要是介绍 Jvm调优-工具总结 。

- [JVM调优工具的使用（jps,jstat,jstack,jmap,jhat）](https://www.yijiyong.com/dev/tool/06-jvmtool.html#jvm%E8%B0%83%E4%BC%98%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%BF%E7%94%A8-jps-jstat-jstack-jmap-jhat)
- [VisualVM可视化工具总结推荐：](https://www.yijiyong.com/dev/tool/06-jvmtool.html#visualvm%E5%8F%AF%E8%A7%86%E5%8C%96%E5%B7%A5%E5%85%B7%E6%80%BB%E7%BB%93%E6%8E%A8%E8%8D%90)
  - [我们在Java的开发中，常常会遇到下面这些问题：](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E6%88%91%E4%BB%AC%E5%9C%A8java%E7%9A%84%E5%BC%80%E5%8F%91%E4%B8%AD-%E5%B8%B8%E5%B8%B8%E4%BC%9A%E9%81%87%E5%88%B0%E4%B8%8B%E9%9D%A2%E8%BF%99%E4%BA%9B%E9%97%AE%E9%A2%98)
- [常见的命令方式jvm工具](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%B8%B8%E8%A7%81%E7%9A%84%E5%91%BD%E4%BB%A4%E6%96%B9%E5%BC%8Fjvm%E5%B7%A5%E5%85%B7)
  - [一，jps命令](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%B8%80-jps%E5%91%BD%E4%BB%A4)
  - [二 jstat命令](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%BA%8C-jstat%E5%91%BD%E4%BB%A4)
  - [三 jstack命令](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%B8%89-jstack%E5%91%BD%E4%BB%A4)
  - [常用命令 jstack PID （切换到对应用户下）](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4-jstack-pid-%E5%88%87%E6%8D%A2%E5%88%B0%E5%AF%B9%E5%BA%94%E7%94%A8%E6%88%B7%E4%B8%8B)
  - [实例找出某个Java进程中最耗费CPU的Java线程并定位堆栈信息，用到的命令有ps、top、printf、jstack、grep。](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%AE%9E%E4%BE%8B%E6%89%BE%E5%87%BA%E6%9F%90%E4%B8%AAjava%E8%BF%9B%E7%A8%8B%E4%B8%AD%E6%9C%80%E8%80%97%E8%B4%B9cpu%E7%9A%84java%E7%BA%BF%E7%A8%8B%E5%B9%B6%E5%AE%9A%E4%BD%8D%E5%A0%86%E6%A0%88%E4%BF%A1%E6%81%AF-%E7%94%A8%E5%88%B0%E7%9A%84%E5%91%BD%E4%BB%A4%E6%9C%89ps%E3%80%81top%E3%80%81printf%E3%80%81jstack%E3%80%81grep%E3%80%82)
  - [四 jmap](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%9B%9B-jmap)
- [参考文章](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)

## [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#jvm%E8%B0%83%E4%BC%98%E5%B7%A5%E5%85%B7%E7%9A%84%E4%BD%BF%E7%94%A8-jps-jstat-jstack-jmap-jhat)JVM调优工具的使用（jps,jstat,jstack,jmap,jhat）

JDK本身提供了很丰富的性能监控工具，除了集成式的visualVM和jConsole外，还有jstat,jstack,jps,jmap,jhat小工具，这些都是性能调优的常用工具

## [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#visualvm%E5%8F%AF%E8%A7%86%E5%8C%96%E5%B7%A5%E5%85%B7%E6%80%BB%E7%BB%93%E6%8E%A8%E8%8D%90)VisualVM可视化工具总结推荐：

JVM优化之VisualVM工具的使用 链接地址 [【JVM优化之VisualVM工具的使用】(opens new window)](https://www.cnblogs.com/wishsaber/p/12411872.html)

[【JVM优化之VisualVM工具的使用】(opens new window)](https://www.cnblogs.com/wishsaber/p/12411872.html)

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E6%88%91%E4%BB%AC%E5%9C%A8java%E7%9A%84%E5%BC%80%E5%8F%91%E4%B8%AD-%E5%B8%B8%E5%B8%B8%E4%BC%9A%E9%81%87%E5%88%B0%E4%B8%8B%E9%9D%A2%E8%BF%99%E4%BA%9B%E9%97%AE%E9%A2%98)我们在Java的开发中，常常会遇到下面这些问题：

1内存不足，2内存泄漏，3锁争用，4线程锁死，5java进程消耗CPU过高等

这些问题出现的时候 大家常常通过重启服务器或者调大内存来临时解决

## [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%B8%B8%E8%A7%81%E7%9A%84%E5%91%BD%E4%BB%A4%E6%96%B9%E5%BC%8Fjvm%E5%B7%A5%E5%85%B7)常见的命令方式jvm工具

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%B8%80-jps%E5%91%BD%E4%BB%A4)一，jps命令

jps主要用来输出JVM中运行的进程状态信息

命令行参数选项：

-q 不输出类名、Jar名和传入main方法的参数

-m 输出传入main方法的参数

-l 输出main类或Jar的全限名

-v 输出传入JVM的参数

最常用的是 jps -l

```
[root@zabbix-agent bin]# jps -l
1217 sun.tools.jps.Jps
1094 org.apache.catalina.startup.Bootstrap

也可以通过 ps -ef|grep java  查看所有Java应用进程1.2.3.4.5.
```

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%BA%8C-jstat%E5%91%BD%E4%BB%A4)二 jstat命令

jstat 它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据

1 类加载统计：

```
[root@zabbix-agent bin]# jstat -class 1094
Loaded  Bytes  Unloaded  Bytes     Time   
  3133  6273.2        0     0.0       1.91

======================
Loaded:加载class的数量
Bytes：所占用空间大小
Unloaded：未加载数量
Bytes:未加载占用空间
Time：时间
======================1.2.3.4.5.6.7.8.9.10.11.
```

2 编译统计

```
[root@zabbix-agent bin]# jstat -compiler 1094
Compiled Failed Invalid   Time   FailedType FailedMethod
    1994      1       0     5.23          1 org/apache/tomcat/util/IntrospectionUtils setProperty

===================
Compiled：编译数量。
Failed：失败数量
Invalid：不可用数量
Time：时间
FailedType：失败类型
FailedMethod：失败的方法
===================1.2.3.4.5.6.7.8.9.10.11.12.
```

3 垃圾回收统计

```
[root@zabbix-agent bin]# jstat -gc 1094
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
832.0  832.0   96.4   0.0    7040.0   6278.2   17372.0    16028.9   20224.0 19493.8 2304.0 2056.7     18    0.141   1      0.030    0.171

====================
S0C : survivor0区的总容量
S1C : survivor1区的总容量
S0U : survivor0区已使用的容量
S1C : survivor1区已使用的容量
EC : Eden区的总容量
EU : Eden区已使用的容量
OC : Old区的总容量
OU : Old区已使用的容量
PC 当前perm的容量 (KB)
PU perm的使用 (KB)
YGC : 新生代垃圾回收次数
YGCT : 新生代垃圾回收时间
FGC : 老年代垃圾回收次数
FGCT : 老年代垃圾回收时间
GCT : 垃圾回收总消耗时间
====================
jstat -gc 1094 3000 10

这个命令意思就是每隔3000ms输出1094的gc情况，一共输出10次

[root@zabbix-agent bin]# jstat -gcutil 1094    已使用空间占总空间的百分比
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  12.48   6.74  92.27  96.39  89.27     19    0.158     1    0.030    0.187

[root@zabbix-agent bin]# jstat -gccapacity 1094    同-gc，不过还会输出Java堆各区域使用到的最大、最小空间 （堆内存统计）
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC 
  5440.0  83264.0   8704.0  832.0  832.0   7040.0    10944.0   166592.0    17372.0    17372.0      0.0 1067008.0  20224.0      0.0 1048576.0   2304.0     19     11.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.
```

更多jstat命令参数 可参看https://www.cnblogs.com/lizhonghua34/p/7307139.html

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E4%B8%89-jstack%E5%91%BD%E4%BB%A4)三 jstack命令

jstack主要用来查看某个Java进程内的线程堆栈信息，jstack可以定位到线程堆栈，根据堆栈信息我们可以定位到具体代码，所以它在JVM性能调优中使用得非常多。

命令行参数选项

-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况

-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4-jstack-pid-%E5%88%87%E6%8D%A2%E5%88%B0%E5%AF%B9%E5%BA%94%E7%94%A8%E6%88%B7%E4%B8%8B)常用命令 jstack PID （切换到对应用户下）

下面通过一个示例说明；

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%AE%9E%E4%BE%8B%E6%89%BE%E5%87%BA%E6%9F%90%E4%B8%AAjava%E8%BF%9B%E7%A8%8B%E4%B8%AD%E6%9C%80%E8%80%97%E8%B4%B9cpu%E7%9A%84java%E7%BA%BF%E7%A8%8B%E5%B9%B6%E5%AE%9A%E4%BD%8D%E5%A0%86%E6%A0%88%E4%BF%A1%E6%81%AF-%E7%94%A8%E5%88%B0%E7%9A%84%E5%91%BD%E4%BB%A4%E6%9C%89ps%E3%80%81top%E3%80%81printf%E3%80%81jstack%E3%80%81grep%E3%80%82)实例找出某个Java进程中最耗费CPU的Java线程并定位堆栈信息，用到的命令有ps、top、printf、jstack、grep。

首先找出应用进程的PID

```
ps -ef|grep java

[root@zabbix-agent bin]# jps -l
1217 sun.tools.jps.Jps
1094 org.apache.catalina.startup.Bootstrap

[root@zabbix-agent bin]# ps -ef|grep java
root       1094      1  0 18:35 pts/0    00:00:17 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/bin/java -Djava.util.logging.config.file=/home/apache-tomcat-8.5.31/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /home/apache-tomcat-8.5.31/bin/bootstrap.jar:/home/apache-tomcat-8.5.31/bin/tomcat-juli.jar -Dcatalina.base=/home/apache-tomcat-8.5.31 -Dcatalina.home=/home/apache-tomcat-8.5.31 -Djava.io.tmpdir=/home/apache-tomcat-8.5.31/temp org.apache.catalina.startup.Bootstrap start1.2.3.4.5.6.7.8.
```

找到应用进程的PID为1094，接着找出该进程最消耗CPU的线程，可以使用ps -Lfp 1094 或者top -Hp 1094来查看该进程中线程的cpu消耗情况

```
[root@zabbix-agent bin]# top -Hp 1094
top - 19:40:51 up  1:52,  1 user,  load average: 0.00, 0.01, 0.05
Threads:  44 total,   0 running,  44 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   998628 total,    84224 free,   690548 used,   223856 buff/cache
KiB Swap:  2097148 total,  2068536 free,    28612 used.   146256 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                         
  1094 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.00 java                                            
  1095 root      20   0 2778656  84996   5324 S  0.0  8.5   0:01.01 java                                            
  1096 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.54 java                                            
  1097 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.00 java                                            
  1098 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.00 java                                            
  1099 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.00 java                                            
  1100 root      20   0 2778656  84996   5324 S  0.0  8.5   0:03.06 java                                            
  1101 root      20   0 2778656  84996   5324 S  0.0  8.5   0:01.46 java                                            
  1102 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.00 java                                            
  1103 root      20   0 2778656  84996   5324 S  0.0  8.5   0:06.76 java                                            
  1104 root      20   0 2778656  84996   5324 S  0.0  8.5   0:00.59 java1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.
```

查看到线程1103的消耗CPU时间是最大的，用命令查看线程的十六进制值

```
[root@zabbix-agent bin]# printf "%x\n" 1103
44f

查询到的值44f 用于下面jstack 定位信息1.2.3.4.
[root@zabbix-agent bin]# jstack 1094|grep 44f
"VM Periodic Task Thread" os_prio=0 tid=0x00007f602c01bc80 nid=0x44f waiting on condition1.2.
```

这样就可以看到VM Periodic Task Thread 是最耗时的类，然后就可以去代码审查代码了。

### [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%9B%9B-jmap)四 jmap

jmap命令参数

```
dump : 生成堆转储快照
finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
heap : 显示Java堆详细信息
histo : 显示堆中对象的统计信息
permstat : to print permanent generation statistics
F : 当-dump没有响应时，强制生成dump快照1.2.3.4.5.6.
```

常用情况，用jmap把进程内存使用情况dump到文件中，再用jhat分析查看

```
jmap -dump:format=b,file=/tmp/dump.dat 1094
生成dump.dat文件1.2.
```

dump出来的文件可以用MAT、VisualVM等工具查看，这里用jhat查看

```
[root@zabbix-agent /]# jmap -dump:format=b,file=/tmp/dump.dat 1094
Dumping heap to /tmp/dump.dat ...
Heap dump file created
[root@zabbix-agent /]# jhat -port 9999 /tmp/dump.dat 
Reading from /tmp/dump.dat...
Dump file created Sat Jun 23 20:34:39 CST 2018
Snapshot read, resolving...
Resolving 274150 objects...
Chasing references, expect 54 dots......................................................
Eliminating duplicate references......................................................
Snapshot resolved.
Started HTTP server on port 9999
Server is ready.1.2.3.4.5.6.7.8.9.10.11.12.13.
```

下面就可以浏览器界面查看了

![wxmp](https://www.yijiyong.com/assets/img/dev/tool/jvm-1.png)

## [#](https://www.yijiyong.com/dev/tool/06-jvmtool.html#%E5%8F%82%E8%80%83%E6%96%87%E7%AB%A0)参考文章

- https://blog.51cto.com/superleedo/2132016
- https://www.cnblogs.com/wishsaber/p/12411872.html
