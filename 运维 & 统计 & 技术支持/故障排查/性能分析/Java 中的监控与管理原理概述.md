# Java 中的监控与管理原理概述



这篇文章是 **Java 性能分析监控与优化**系列的第一篇文章，原本是计划系统的介绍 Java 性能分析方式和流行的监控工具，但是提笔之后意识到，只介绍分析方式和监控工具过于浅尝辄止了。如果只会使用某个工具而不知道背后的实现原理，总觉得有种陌生感，我想你们也是一样，所以多了这篇文章。

![文章目录](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211127214201.png)

## [#](https://www.wdbyte.com/java/monitoring.html#java-se-%E7%9B%91%E6%8E%A7%E7%AE%A1%E7%90%86%E5%8A%9F%E8%83%BD)Java SE 监控管理功能

这篇文章介绍 Java Standard Edition（Java SE）平台提供的监控和管理技术 - **JMX**（Java Management Extensions） 技术。

Java SE 平台本身就提供了用于监控和管理服务的实用性功能模块，按功能来说主要分为下面四类：

- Java 监控和管理 API
- Java 虚拟机检测
- Java 管理扩展技术（JMX）
- Java 监控和管理的工具

这篇文章会介绍这四个部分的相关知识，旨在了解 Java SE 监控与管理的相关功能，对其中的相关概念有个理解。

## [#](https://www.wdbyte.com/java/monitoring.html#java-%E7%9B%91%E6%8E%A7%E5%92%8C%E7%AE%A1%E7%90%86-api)Java 监控和管理 API

Java SE 中包含了用于监控和管理的（`java.lang.management`）API，通过这些 API 可以实现应用程序的自我监控，此 API 主要提供了以下信息的访问：

- 类加载相关。
- JVM 相关，如运行时间、系统环境变量、用户输入参数。
- 线程相关，如线程状态，线程的统计信息、线程的堆栈等。
- 内存使用情况。
- GC 情况。
- 死锁检测。
- 操作系统信息。

下图是 Java 17 中的 `java.management` 模块。

![java.lang.management](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/20211126142407.png)

JConsole 就是通过访问这些管理 API 提供的数据，绘制了监控的界面版。

## [#](https://www.wdbyte.com/java/monitoring.html#java-%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%9B%91%E6%B5%8B)Java 虚拟机监测

上面说到 Java SE 中已经内置了开箱即用的监控和管理功能，通过这些功能可以实现程序的自我监测，Java 默认已经实现了对 Java 虚拟机相关信息的监测，在 Java 监控和管理 API 部分也列举了 API 可以监测的部分内容，那么怎么使用呢？

下面通过一个简单的示例，演示如何通过监控管理 API 获取系统信息、编译器信息、内存信息以及垃圾收集器信息。

```
package com.wdbyte;

import java.lang.management.CompilationMXBean;
import java.lang.management.GarbageCollectorMXBean;
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.MemoryManagerMXBean;
import java.lang.management.MemoryUsage;
import java.lang.management.OperatingSystemMXBean;
import java.util.List;
import java.util.stream.Collectors;

public class JavaManagement {

    public static void main(String[] args) {
        OperatingSystemMXBean operatingSystemMXBean = ManagementFactory.getOperatingSystemMXBean();
        String osName = operatingSystemMXBean.getName();
        String osVersion = operatingSystemMXBean.getVersion();
        int processors = operatingSystemMXBean.getAvailableProcessors();
        System.out.println(String.format("操作系统：%s，版本：%s，处理器：%d 个", osName, osVersion, processors));

        CompilationMXBean compilationMXBean = ManagementFactory.getCompilationMXBean();
        String compilationMXBeanName = compilationMXBean.getName();
        System.out.println("编译系统：" + compilationMXBeanName);

        MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapMemoryUsage = memoryMXBean.getHeapMemoryUsage();
        long max = heapMemoryUsage.getMax();
        long used = heapMemoryUsage.getUsed();
        System.out.println(String.format("使用内存：%dMB/%dMB", used / 1024 / 1024, max / 1024 / 1024));

        List<GarbageCollectorMXBean> gcMXBeans = ManagementFactory.getGarbageCollectorMXBeans();
        String gcNames = gcMXBeans.stream()
            .map(MemoryManagerMXBean::getName)
            .collect(Collectors.joining(","));
        System.out.println("垃圾收集器：" + gcNames);
    }
}
```



运行时指定了内存为 100MB（`-Xms100M -Xmx100M`），得到如下结果。

```
操作系统：Mac OS X，版本：11.6，处理器：12 个
编译系统：HotSpot 64-Bit Tiered Compilers
使用内存：2MB/100MB
垃圾收集：G1 Young Generation,G1 Old Generation
```



注意

细看代码可以发现其中很多类都是以 **MXBean** 结尾，这是什么意思呢？

## [#](https://www.wdbyte.com/java/monitoring.html#java-%E7%AE%A1%E7%90%86%E6%89%A9%E5%B1%95%E6%8A%80%E6%9C%AF-jmx)Java 管理扩展技术（JMX）

在 Java 虚拟机监测中的代码示例中，可以看到很多命名以 **MXBean** 结尾的类，这里已经涉及到了 **JMX**（Java Management Extensions） 技术。

**JMX** 技术提供了一种简单、标准的方式来管理资源，如操作系统、虚拟机信息、内存状态、线程信息等，这些统称为被管理的资源。而且 **JMX** 是可以动态的，所以可以使用 **JMX** 技术来监测和管理各种资源。可以使用 JMX 技术来监测 Java 虚拟机状态，也可以使用 **JMX** 技术构建自己的需要管理的资源。

JMX 技术只有资源定义那么简单吗？不是的。JMX 规范了 Java 中资源定义的方式、资源管理的方式、监控和管理的体系结构、具体实现的设计模式、监控和管理的相关 API 以及**用于网络的远程监控服务**（RMI），这一系列功能统称为 JMX 技术。是 Java SE 平台的标准部分。

上面多次提到了**管理资源**，那么如何定义一个资源呢？JMX 技术给出了资源定义的体系结构和设计模式，在 **JMX** 中，通过定义一个被称为 **MBean** 或 **MXBean** 的 Java 对象来表示要管理指定的资源，资源定义的 Java 类名必须以 **MBean** 或 **MXBean** 结尾。

下图是 Java 17 中的以 MXBean 结尾的资源定义类，通过命名可以看出每个类代表了什么资源。

![Java 中的 MXbean](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/20211126212731.png)

这篇文章主要是介绍 Java SE 中的监控与管理功能，让大家对 Java 中的监控与管理背后的原理和概念有一个具体的认识，所以 MBean 和 MXBean 的具体设计实现方式不是这篇文章的重点，这里不过多介绍，会放到下一篇独立的 JMX 技术文章中介绍。

## [#](https://www.wdbyte.com/java/monitoring.html#java-%E7%9B%91%E6%8E%A7%E5%92%8C%E7%AE%A1%E7%90%86%E7%9A%84%E5%B7%A5%E5%85%B7)Java 监控和管理的工具

JMX 技术中提到 JMX 不仅提供了监控和管理的 API ，还提供了用于网络远程管理的服务，可以使用 JMX 相关监控管理工具，通过网络远程连接到正在运行 Java 虚拟机，监控其运行状态，Java 中集成的 `jconsole` 就是这样一款工具。

本地随意启动一个可以持续运行的 Java 程序用作被监测对象，如果你已经配置好 Java 环境变量，可以直接通过 `jconsole` 启动工具。

```
 $ jconsole
```

1  

启动后的 `jconsole` 已经列出了本地正在运行的 Java 程序，选择自己想要监测的进行进行监测。

![Jconsole 界面](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/20211127154455.png)

连接成功后可以看到当前 Java 进程的资源占用情况。

![JConsole 监控](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/20211127154215.png)

在 MBean 页面中，可以看到各种已经被定义的资源的具体情况。

![Jconsole MBean 情况](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/20211127154332.png)

Jconsole 是一款强大的图形界面 JMX 管理工具，不仅可以连接本地 Java 程序，还可以通过网络监控远程的 Java 程序运行状态，不过不是此篇文章重点，不在详细描述。

一如既往，当前文章中的代码示例都存放在 [github.com/niumoo/JavaNotes (opens new window)](https://github.com/niumoo/JavaNotes/tree/master/core-java-modules/).

**当前系列：**

1. [Java 中的监控与管理原理概述(opens new window)](https://www.wdbyte.com/java/monitoring.html)
2. [使用 JMX 监控和管理 Java 程序(opens new window)](https://www.wdbyte.com/java/jmx.html)

**参考：**

- [https://docs.oracle.com/en/java/javase/17/jmx/(opens new window)](https://docs.oracle.com/en/java/javase/17/jmx/)
- [Java Platform, Standard Edition Monitoring and Management Guide, Release 17](https://docs.oracle.com/en/java/javase/17/management/)
