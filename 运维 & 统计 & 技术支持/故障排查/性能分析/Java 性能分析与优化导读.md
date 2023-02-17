# Java 性能分析与优化导读





回顾最近半年的工作，发现遇到了不少 **Java 运行时问题**，这些问题可能是内存溢出、CPU 飙升、延迟增加、队列堵塞等等。之前遇到，现在遇到，未来一定还会遇到，索性把 Java 相关性能分析工具和手段进行一个系统梳理，给大家一个参考，也给未来的自己一个参考。

下面是整理的常见的 **Java 性能分析工具**，知识的梳理需要结构化，做成了一个思维导图，计划一一介绍它们的使用方式和使用场景。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211127215041.png)

- *Java 性能分析优化思维导图*

性能分析优化是一个优化资源配置的过程，计算机系统中的 CPU、内存、磁盘以及网络资源是有限的，性能分析优化也就是通过测量系统的各个环节的性能，找到性能的瓶颈，然后优化之，所以如何测量找到问题，是关键所在，这些工具都可以帮助我们更好的定位。

## [#](https://www.wdbyte.com/java/performance.html#java-%E4%B8%AD%E7%9A%84%E7%AE%A1%E7%90%86%E7%9B%91%E6%8E%A7%E5%8A%9F%E8%83%BD)Java 中的管理监控功能

### [#](https://www.wdbyte.com/java/performance.html#%E6%A6%82%E8%BF%B0)概述

这篇文章介绍 Java Standard Edition（Java SE）平台提供的监控和管理技术 - **JMX** 技术。

![文章目录](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211127214201.png)

相关文章：[Java 中的管理监控功能(opens new window)](https://www.wdbyte.com/java/monitoring.html)

### [#](https://www.wdbyte.com/java/performance.html#jmx)JMX

此篇文章介绍 Java JMX 技术的相关概念和具体的使用方式。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204223343.png)

相关文章：[使用 JMX 监控和管理 Java 程序(opens new window)](https://www.wdbyte.com/java/jmx.html)

## [#](https://www.wdbyte.com/java/performance.html#java-%E6%80%A7%E8%83%BD%E7%9B%91%E6%8E%A7%E5%BC%80%E6%BA%90%E5%B7%A5%E5%85%B7)Java 性能监控开源工具

### [#](https://www.wdbyte.com/java/performance.html#arthas)Arthas

**Arthas** 是 `Alibaba` 在 2018 年 9 月开源的 **Java 诊断**工具。支持 `JDK6+`， 采用命令行交互模式，提供 `Tab` 自动补全，可以方便的定位和诊断线上程序运行问题。截至 2021 年 11 月 24 日，已经收获 `Star` 27000+。

在使用 **Arthas** 之前，当遇到 Java 线上问题时，如 CPU 飙升、负载突高、内存溢出等问题，你需要查命令，查网络，然后 jps、jstack、jmap、jhat、jstat、hprof 等一通操作。最终焦头烂额，还不一定能查出问题所在。而现在，大多数的常见问题你都可以使用 **Arthas** 轻松定位，迅速解决，及时止损，准时下班。

介绍文章：[Arthas - Java 线上问题定位处理的终极利器(opens new window)](https://www.wdbyte.com/2019/11/arthas/)

### [#](https://www.wdbyte.com/java/performance.html#tprofiler)Tprofiler

### [#](https://www.wdbyte.com/java/performance.html#profiler4j)Profiler4J

## [#](https://www.wdbyte.com/java/performance.html#java-%E6%80%A7%E8%83%BD%E7%9B%91%E6%8E%A7%E5%92%8C%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7)Java 性能监控和管理工具

### [#](https://www.wdbyte.com/java/performance.html#jmc-java-mission-control)JMC（Java Mission Control）

### [#](https://www.wdbyte.com/java/performance.html#jfr-java-flight-recorder)JFR（Java Flight Recorder）

### [#](https://www.wdbyte.com/java/performance.html#jconsole)jconsole

### [#](https://www.wdbyte.com/java/performance.html#jvisualvm)jVisualVM

## [#](https://www.wdbyte.com/java/performance.html#java-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E5%91%BD%E4%BB%A4%E5%B7%A5%E5%85%B7)Java 性能分析命令工具

### [#](https://www.wdbyte.com/java/performance.html#jmap-%E5%86%85%E5%AD%98%E5%88%86%E6%9E%90)jmap（内存分析）

### [#](https://www.wdbyte.com/java/performance.html#jhat)jhat

### [#](https://www.wdbyte.com/java/performance.html#jstack-%E5%A0%86%E6%A0%88%E5%88%86%E6%9E%90)jstack（堆栈分析）

### [#](https://www.wdbyte.com/java/performance.html#jstat-gc)jstat（GC

### [#](https://www.wdbyte.com/java/performance.html#jinfo)jinfo

### [#](https://www.wdbyte.com/java/performance.html#jcmd)jcmd

## [#](https://www.wdbyte.com/java/performance.html#java-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E5%95%86%E4%B8%9A%E5%B7%A5%E5%85%B7)Java 性能分析商业工具

### [#](https://www.wdbyte.com/java/performance.html#jprofiler)Jprofiler

### [#](https://www.wdbyte.com/java/performance.html#yourkit-java-profiler)YourKit Java Profiler

## [#](https://www.wdbyte.com/java/performance.html#jmh-%E6%80%A7%E8%83%BD%E5%9F%BA%E5%87%86%E6%B5%8B%E8%AF%95)JMH 性能基准测试

JMH 是一个 Java 工具，用于构建、运行和分析 Java 程序和其他基于 JVM 的编写的程序的基准性能。
