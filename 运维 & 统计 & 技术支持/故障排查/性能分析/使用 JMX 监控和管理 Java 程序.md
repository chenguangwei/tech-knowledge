# 使用 JMX 监控和管理 Java 程序



此篇文章介绍 Java JMX 技术的相关概念和具体的使用方式。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204223343.png)

> 当前文章属于 [Java 性能分析优化 (opens new window)](https://www.wdbyte.com/java/performance.html)系列文章，点击可以查看所有文章。
> 
> 系列链接：[https://www.wdbyte.com/java/performance.html(opens new window)](https://www.wdbyte.com/java/performance.html)

## [#](https://www.wdbyte.com/java/jmx.html#_1-jmx-%E6%98%AF%E4%BB%80%E4%B9%88)1. JMX 是什么？

Java Management Extensions（**JMX**）技术是 Java SE 平台的标准功能，提供了一种简单的、标准的监控和管理资源的方式，对于如何定义一个资源给出了明确的结构和设计模式，主要用于监控和管理 Java 应用程序运行状态、设备和资源信息、Java 虚拟机运行情况等信息。 **JMX** 是可以动态的，所以也可以在资源创建、安装、实现时进行动态监控和管理，JDK 自带的 jconsole 就是使用 **JMX** 技术实现的监控工具。

使用 **JMX** 技术时，通过定义一个被称为 **MBean** 或 **MXBean** 的 Java 对象来表示要管理指定的资源，然后可以把资源信息注册到 **MBean Server** 对外提供服务。MBean Server 充当了对外提供服务和对内管理 MBean 资源的代理功能，如此优雅的设计让 MBean 资源管理和 MBean Server 代理完全独立开，使之可以自由的控制 MBean 资源信息。

JMX 不仅仅用于本地管理，JMX Remote API 为 JMX 添加了远程功能，使之可以通过网络远程监视和管理应用程序。

## [#](https://www.wdbyte.com/java/jmx.html#_2-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8-jmx-%E6%8A%80%E6%9C%AF)2. 为什么使用 JMX 技术？

JMX 技术为 Java 开发者提供了一种简单、灵活、标准的方式来监测 Java 应用程序，得益于相对独立的架构设计，使 JMX 可以平滑的集成到各种监控系统之中。

下面列举几项 JMX 的具体优点：

1. 开箱即用的监控功能，JMX 是 Java SE 的标准部分，提供了资源管理、服务托管、远程监控等管理基础功能，都可以直接启用。
2. JMX 技术提供了一种通用的、标准的资源、系统、应用程序、网络的管理方式，不仅可以本地使用、远程使用；还可以扩展到其他场景，如 Java EE 应用等。
3. JMX 技术提供了对 JVM 状态的监测功能，JMX 已经内置了对 JVM 的监测功能，并且可以监控和管理 JVM，十分方便。
4. JMX 架构设计优秀，组件化的设计可以自由的扩展。
5. JMX 技术严格遵守 Java 现有规范如 JNDI 规范。
6. JMX 可以自由的与其他管理解决方案集成，得益于开放的 JMX API，可以通过 web 服务管理 JMX 中的资源。

## [#](https://www.wdbyte.com/java/jmx.html#_3-jmx-%E7%9A%84%E6%8A%80%E6%9C%AF%E6%9E%B6%E6%9E%84)3. JMX 的技术架构

JMX 技术架构主要有资源管理（MBean/MXBean）模块，资源代理模块（MBean Server），远程管理模块（Remote API）组成 ，下面的图片来自维基百科，很好的展示了三个模块之间的关系。

![图片来自维基百科](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/draw/JMX_Architecture.svg)

JMX 技术架构图（来自维基百科）

### [#](https://www.wdbyte.com/java/jmx.html#_3-1-%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86-mbean)3.1. 资源管理 MBean

资源管理在架构中标识为资源探测层（Probe Level），在 JMX 中， 使用 **MBean** 或 **MXBean** 来表示一个资源（下面简称 MBean），访问和管理资源也都是通过 MBean，所以 MBean 往往包含着资源的**属性和操作方法**。

JMX 已经对 JVM 进行了多维度资源检测，所以可以轻松启动 JMX 代理来访问内置的 JVM 资源检测，从而通过 JMX 技术远程监控和管理 JVM。

下面列举 JMX 对 JVM 的资源检测类，都可以直接使用。

| 资源接口                   | 管理的资源 | Object Name                                          | VM 中的实例个数 |
| ---------------------- | ----- | ---------------------------------------------------- | --------- |
| ClassLoadingMXBean     | 类加载   | java.lang:type= ClassLoading                         | 1 个       |
| CompilationMXBean      | 汇编系统  | java.lang:type= Compilation                          | 0 个或 1 个  |
| GarbageCollectorMXBean | 垃圾收集  | java.lang:type= GarbageCollector, name=collectorName | 1 个或更多    |
| LoggingMXBean          | 日志系统  | java.util.logging:type =Logging                      | 1 个       |
| MemoryManagerMXBean    | 内存池   | java.lang: typeMemoryManager, name=managerName       | 1 个或更多    |
| MemoryPoolMXBean       | 内存    | java.lang: type= MemoryPool, name=poolName           | 1 个或更多    |
| MemoryMXBean           | 内存系统  | java.lang:type= Memory                               | 1 个       |
| OperatingSystemMXBean  | 操作系统  | java.lang:type= OperatingSystem                      | 1 个       |
| RuntimeMXBean          | 运行时系统 | java.lang:type= Runtime                              | 1 个       |
| ThreadMXBean           | 线程系统  | java.lang:type= Threading                            | 1 个       |

下面的代码示例演示了使用 JMX 检测 JVM 某些信息的代码示例。

```
package com.wdbyte.jmx;

import java.lang.management.CompilationMXBean;
import java.lang.management.GarbageCollectorMXBean;
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.MemoryManagerMXBean;
import java.lang.management.MemoryUsage;
import java.lang.management.OperatingSystemMXBean;
import java.util.List;
import java.util.stream.Collectors;

/** * JMX JVM * * @author https://www.wdbyte.com */
public class JavaManagementExtensions {

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

运行可以得到如下结果：

```
操作系统：Mac OS X，版本：11.6，处理器：12 个
编译系统：HotSpot 64-Bit Tiered Compilers
使用内存：3MB/4096MB
垃圾收集器：G1 Young Generation,G1 Old Generation
```

### [#](https://www.wdbyte.com/java/jmx.html#_3-2-%E8%B5%84%E6%BA%90%E4%BB%A3%E7%90%86-mbean-server)3.2. 资源代理 MBean Server

资源代理 MBean Server 是 MBean 资源的代理，通过 MBean Server 可以让 MBean 资源用于远程管理， MBean 资源和 MBean Server 往往都是在同一个 JVM 中，但这不是必须的。

想要 MBean Server 可以管理 MBean 资源，首先要把资源注册到 MBean Server，任何符合 JMX 的 MBean 资源都可以进行注册，最后 MBean Server 会暴露一个远程通信接口对外提供服务。

### [#](https://www.wdbyte.com/java/jmx.html#_3-3-jmx-%E8%BF%9C%E7%A8%8B%E7%AE%A1%E7%90%86)3.3. JMX 远程管理

可以通过网络协议访问 JMX API，如 HTTP 协议、SNMP（网络管理协议）协议、RMI 远程调用协议等，JMX 技术默认实现了 RMI 远程调用协议。

受益于资源管理 MBean 的充分解耦，可以轻松的把资源管理功能扩展到其他协议，如通过 HTTP 在网页端进行管理。

## [#](https://www.wdbyte.com/java/jmx.html#_4-jmx-%E7%9A%84%E5%85%B7%E4%BD%93%E4%BD%BF%E7%94%A8)4. JMX 的具体使用

在资源管理 MBean 部分已经演示了使用 JMX 获取 JVM 运行信息，那么如果想要自定义一个资源 MBean 呢？

下面通过一个例子，模拟一个内存资源 MBean，最后对它进行远程管理。

### [#](https://www.wdbyte.com/java/jmx.html#_4-1-%E7%BC%96%E5%86%99%E8%B5%84%E6%BA%90%E7%AE%A1%E7%90%86-mbean)4.1. 编写资源管理 MBean

MBean 的编写必须遵守 JMX 的设计规范，MBean 很像一个特殊的 Java Bean，它需要一个接口和一个实现类。**MBean 资源接口总是以 MBean 或者 MXBean 结尾**，**实现类则要以接口去掉 MBean 或 MXBean 之后的名字来命名**。

编写一个内存资源管理 MBean 接口，定义如下：

```
package com.wdbyte.jmx;

/** * @author https://www.wdbyte.com */
public interface MyMemoryMBean {

    long getTotal();

    void setTotal(long total);

    long getUsed();

    void setUsed(long used);

    String doMemoryInfo();
}
```



然后实现这个接口：

```
package com.wdbyte.jmx;

/** * @author https://www.wdbyte.com */
public class MyMemory implements MyMemoryMBean {

    private long total;
    private long used;

    @Override
    public long getTotal() {
        return total;
    }

    @Override
    public void setTotal(long total) {
        this.total = total;
    }

    @Override
    public long getUsed() {
        return used;
    }

    @Override
    public void setUsed(long used) {
        this.used = used;
    }

    @Override
    public String doMemoryInfo() {
        return String.format("使用内存: %dMB/%dMB", used, total);
    }

}
```

提示

这个例子在 `MyMemory.java` 中只有两个 `long` 基本类型属性，所以接口是以 MBean 结尾。如果资源实现类中的属性是自定义实体类的引用，那么接口就需要以 **MXBean** 结尾。

这样就完成了线程数量资源 MBean 的创建，其中 `total` 和 `used` 是资源属性，`doMemoryInfo` 是资源操作方法。

### [#](https://www.wdbyte.com/java/jmx.html#_4-2-%E6%B3%A8%E5%86%8C%E8%B5%84%E6%BA%90%E5%88%B0-mbean-server)4.2. 注册资源到 MBean Server

通过上面的 JMX 架构图，我们知道 MBean 资源需要注册到 MBean Server 进行代理才可以暴露给外部进行调用，所以我们想要通过远程管理我们自定义的 MyMemory 资源，需要先进行资源代理。

```
package com.wdbyte.jmx;

import java.lang.management.ManagementFactory;

import javax.management.InstanceAlreadyExistsException;
import javax.management.MBeanRegistrationException;
import javax.management.MBeanServer;
import javax.management.MalformedObjectNameException;
import javax.management.NotCompliantMBeanException;
import javax.management.ObjectName;

/** * @author https://www.wdbyte.com */
public class MyMemoryManagement {

    public static void main(String[] args) throws MalformedObjectNameException, NotCompliantMBeanException,
        InstanceAlreadyExistsException, MBeanRegistrationException, InterruptedException {
        // 获取 MBean Server
        MBeanServer platformMBeanServer = ManagementFactory.getPlatformMBeanServer();
        MyMemory myMemory = new MyMemory();
        myMemory.setTotal(100L);
        myMemory.setUsed(20L);
        // 注册
        ObjectName objectName = new ObjectName("com.wdbyte.jmx:type=myMemory");
        platformMBeanServer.registerMBean(myMemory, objectName);

        while (true) {
            // 防止进行退出
            Thread.sleep(3000);
            System.out.println(myMemory.doMemoryInfo());
        }
    }
}
```

启动后可以看到控制台每隔三秒打印我们自定义的内存信息。

```
使用内存: 20MB/100MB
使用内存: 20MB/100MB
```



提示

不加任何 JVM 参数启动 Java 程序，JMX 只能在当前机器访问，如果想要通过网络在真正的远程访问，那么在启动时需要指定当前机器 ip 和开放的端口。

```
$ java -Dcom.sun.management.jmxremote=true \  # 开启远程访问
-Dcom.sun.management.jmxremote.port=8398 \        # 自定义 JMX 端口
-Dcom.sun.management.jmxremote.ssl=false \        # 是否使用 SSL 协议，生产环境一定要开启
-Dcom.sun.management.jmxremote.authenticate=false \ # 是否需要认证，生产环境一定要开启
-Djava.rmi.server.hostname=150.158.2.56 YourClass.java # 当前机器 ip
```



### [#](https://www.wdbyte.com/java/jmx.html#_4-3-%E8%BF%9C%E7%A8%8B%E7%AE%A1%E7%90%86-jconsole)4.3. 远程管理 jconsole

`jconsole` 是 Java 自带的基于 JMX 技术的监控管理工具，如果已经配置了 JDK 环境变量，可以直接控制台通过 `jconsole` 命令启动。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204221126.png)

启动 jconsole 后会列出当前机器上的 Java 进行，这里选择自己要监控的 Java 进程进行监控，连接后会提示不安全的协议，是因为 Java 程序默认启动是不会配置 HTTPS 协议的原因。

连接后可以看到多维度的 JVM 监控信息，这些信息都是通过读取 JVM 资源 MBean 信息得到的。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204220734.png)

在下面这个页面列举了线程信息，注意最下面的线程信息，可以看到 `RMI TCP` 线程，这里也证明了 JMX 默认通过 RMI 协议进行远程管理。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204220802.png)

在 MBean 页面可以浏览所有可管理的 MBean 信息，也可以看到我们自定义的 `com.wdbyte.jmx` 中的内存信息，甚至可以直接修改其中的 `used` 变量。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204221005.png)

修改后控制台日志立即发生变化，可以看到已经修改成功。

```
使用内存: 20MB/100MB
使用内存: 20MB/100MB
使用内存: 20MB/100MB
使用内存: 30MB/100MB
```



在操作中可以调用 `doMemoryInfo` 方法，调用后可以看到返回值中使用内存已经由启动时的 20MB 更新为 30MB。

![](https://wdbyte-img.oss-cn-hangzhou.aliyuncs.com/git/2021/20211204221055.png)

一如既往，当前文章中的代码示例都存放在 [github.com/niumoo/JavaNotes (opens new window)](https://github.com/niumoo/JavaNotes/tree/master/core-java-modules/).
