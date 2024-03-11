# 【Arthas问题排查集】谁调用了System.exit/System.gc?

我们有时候可能会遇到这样的问题，进程莫名其妙的退出了，或者是发生了GC，通过日志或者是其他办法发现是有人调用了`System.gc/System.exit`，但是确不知道是谁干的。

如何找出这个罪魁祸首呢？一般来说，可以通过一段Btrace脚本来解决

```
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class TraceSystemGCCall {
  @OnMethod(
    clazz="/.*/",
    method="/.*/",
    location=@Location(
      value=Kind.CALL,
      clazz="java.lang.System",
      method="gc"
    )
  )
  public static void onSystemGC() {
    jstack();
  }
}
```

类似这样的脚本（不保证能正常执行啊。。）经常容易写错，导致各种问题，有没有更好的办法呢？

今天我们来分享下，如何通过Arthas排查这类问题。

这里我们假设你已经了解下载，安装，启动Arthas的步骤。

第一步，由于`java.lang.System`是JDK自带的类，Arthas默认关闭了对JDK类的自带类的增强，需要通过`options`命令打开。

```
$ options unsafe true
 NAME    BEFORE-VALUE  AFTER-VALUE
-----------------------------------
 unsafe  false         true
```

第二步，使用`stack`命令，观察谁调用了`java.lang.System#gc`

```
$ stack java.lang.System gc
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 81 ms.
stack java.lang.System gc
```

然后，等待触发即可！

```
$ stack sun.management.MemoryImpl gc
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 27 ms.
stack sun.management.MemoryImpl gc
thread_name=RMI TCP Connection(5)-30.5.50.222;id=29;is_daemon=true;priority=9;TCCL=sun.misc.Launcher$AppClassLoader@14dad5dc
    @sun.management.MemoryImpl.gc()
        at sun.reflect.NativeMethodAccessorImpl.invoke0(NativeMethodAccessorImpl.java:-2)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at sun.reflect.misc.Trampoline.invoke(MethodUtil.java:71)
        at sun.reflect.GeneratedMethodAccessor22.invoke(null:-1)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at sun.reflect.misc.MethodUtil.invoke(MethodUtil.java:275)
        at com.sun.jmx.mbeanserver.ConvertingMethod.invokeWithOpenReturn(ConvertingMethod.java:193)
        at com.sun.jmx.mbeanserver.ConvertingMethod.invokeWithOpenReturn(ConvertingMethod.java:175)
        at com.sun.jmx.mbeanserver.MXBeanIntrospector.invokeM2(MXBeanIntrospector.java:117)
        at com.sun.jmx.mbeanserver.MXBeanIntrospector.invokeM2(MXBeanIntrospector.java:54)
        at com.sun.jmx.mbeanserver.MBeanIntrospector.invokeM(MBeanIntrospector.java:237)
        at com.sun.jmx.mbeanserver.PerInterface.invoke(PerInterface.java:138)
        at com.sun.jmx.mbeanserver.MBeanSupport.invoke(MBeanSupport.java:252)
        at javax.management.StandardMBean.invoke(StandardMBean.java:405)
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819)
        at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801)
        at javax.management.remote.rmi.RMIConnectionImpl.doOperation(RMIConnectionImpl.java:1470)
        at javax.management.remote.rmi.RMIConnectionImpl.access$300(RMIConnectionImpl.java:76)
        at javax.management.remote.rmi.RMIConnectionImpl$PrivilegedOperation.run(RMIConnectionImpl.java:1311)
        at javax.management.remote.rmi.RMIConnectionImpl.doPrivilegedOperation(RMIConnectionImpl.java:1403)
        at javax.management.remote.rmi.RMIConnectionImpl.invoke(RMIConnectionImpl.java:832)
        at sun.reflect.GeneratedMethodAccessor29.invoke(null:-1)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:497)
        at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:323)
        at sun.rmi.transport.Transport$1.run(Transport.java:200)
        at sun.rmi.transport.Transport$1.run(Transport.java:197)
        at java.security.AccessController.doPrivileged(AccessController.java:-2)
        at sun.rmi.transport.Transport.serviceCall(Transport.java:196)
        at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:568)
        at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(TCPTransport.java:826)
        at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$255(TCPTransport.java:683)
        at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler$$Lambda$2/1638002256.run(null:-1)
        at java.security.AccessController.doPrivileged(AccessController.java:-2)
        at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:682)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
```

> 这里有点偷懒，测试用的是jconsole的`performace gc`这个功能，背后调用的是`sun.management.MemoryImpl`的gc方法。效果是一样的。

那如果这个命令很长时间都不执行，我们不可能一直在那里等着呀，这种时候怎么办呢？

这时，我们可以使用arthas的`批处理模式`, 将arthas的命令写成一个脚本来执行。

具体方法是，第一步，准备一个脚本，例如`system_gc.as`，扩展名任意，这里为了便于记忆，采用了`.as`的扩展名，内容如下：

```
➜  tmp cat system_gc.as
options unsafe true
stack java.lang.System gc -n 1
```

其实就是把上面的两条命令放在一个文件里面而已。`务必注意`这里的`-n`，表示这个命令只执行一次，然后退出。如果不指定，Arthas 2.0会一直卡在那里，永远不退出，Arthas 3.0默认执行`100`次才会退出。在写批处理脚本的时候，请务必加上`-n`参数。

然后就可以执行这个脚本了, 默认脚本执行的结果会输出到标准输出，我们可以通过重定向，将输出持久化，最后通过最后一个`&`放到后台运行。

```
./as.sh -b -f system_gc.as 59863 > system_gc.out 2>&1 &
```

然后，在合适的时候去查看文件的结果即可。

```
cat system_gc.out
```

这种情况还有一个问题，就是如果我们的ssh session断了，这个进程就会退出。

如何保持后台运行呢？这里介绍2种办法：

1. 用`nohup`让这个进程不响应`SIGHUP`信号

```
nohup ./as.sh -b -f system_gc.as 59863 > system_gc.out 2>&1 &
```

1. 终端模拟类命令。`screen`或者`tmux`这类命令在线上其实非常有用，可以避免每次ssh连接断掉之后，原来连接中运行进程和数据丢失。具体用法如下:

```
sudo yum install -b current screen
screen
./as.sh -b -f system_gc.as 59863 > system_gc.out 2>&1
```

这时候可以去干别的事情了，即便网络断掉也没有问题。过一段时间之后，要查看之前的结果。如果连接断掉了，只需要输入以下命令：

```
screen -r
```

就可以恢复原来的session，并查看结果。

PS:最新版本的Arthas已经支持通过管道把结果保存到文件当中。详细的文档可见：https://alibaba.github.io/arthas/async.html