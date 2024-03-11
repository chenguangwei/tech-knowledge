# 使用Arthas排查线上应用日志打满问题

## 现象

在应用的 `service_stdout.log`里一直输出下面的日志，直接把磁盘打满了：

```
23:07:34.441 [TAIRCLIENT-1-thread-1] DEBUG io.netty.channel.nio.NioEventLoop - Selector.select() returned prematurely 14 times in a row.
23:07:34.460 [TAIRCLIENT-1-thread-3] DEBUG io.netty.channel.nio.NioEventLoop - Selector.select() returned prematurely 3 times in a row.
23:07:34.461 [TAIRCLIENT-1-thread-4] DEBUG io.netty.channel.nio.NioEventLoop - Selector.select() returned prematurely 3 times in a row.
23:07:34.462 [TAIRCLIENT-1-thread-5] DEBUG io.netty.channel.nio.NioEventLoop - Selector.select() returned prematurely 3 times in a row.
```

`service_stdout.log`是进程标准输出的重定向，可以初步判定是tair插件把日志输出到了stdout里。

尽管有了初步的判断，但是具体logger为什么会打到stdout里，还需要进一步排查，常见的方法可能是本地debug。

下面介绍利用arthas直接在线上定位问题的过程，主要使用`sc`和`getstatic`命令。

- https://alibaba.github.io/arthas/sc.html
- https://alibaba.github.io/arthas/getstatic.html

## 定位logger的具体实现

日志是`io.netty.channel.nio.NioEventLoop`输出的，到netty的代码里查看，发现是DEBUG级别的输出：

- https://github.com/netty/netty/blob/netty-4.0.35.Final/transport/src/main/java/io/netty/channel/nio/NioEventLoop.java#L673

然后用arthas的`sc`命令来查看具体的`io.netty.channel.nio.NioEventLoop`是从哪里加载的。

```
class-info        io.netty.channel.nio.NioEventLoop
 code-source       file:/opt/app/plugins/tair-plugin/lib/netty-all-4.0.35.Final.jar!/
 name              io.netty.channel.nio.NioEventLoop
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
 isLocalClass      false
 isMemberClass     false
 isPrimitive       false
 isSynthetic       false
 simple-name       NioEventLoop
 modifier          final,public
 annotation
 interfaces
 super-class       +-io.netty.channel.SingleThreadEventLoop
                     +-io.netty.util.concurrent.SingleThreadEventExecutor
                       +-io.netty.util.concurrent.AbstractScheduledEventExecutor
                         +-io.netty.util.concurrent.AbstractEventExecutor
                           +-java.util.concurrent.AbstractExecutorService
                             +-java.lang.Object
 class-loader      +-tair-plugin's ModuleClassLoader
 classLoaderHash   73ad2d6
```

可见，的确是从tair插件里加载的。

查看NioEventLoop的代码，可以发现它有一个`logger`的field：

```
public final class NioEventLoop extends SingleThreadEventLoop {

    private static final InternalLogger logger = InternalLoggerFactory.getInstance(NioEventLoop.class);
```

使用arthas的`getstatic`命令来查看这个`logger`具体实现类是什么（使用`-c`参数指定classloader）：

```
$ getstatic -c 73ad2d6 io.netty.channel.nio.NioEventLoop logger 'getClass().getName()'
field: logger
@String[io.netty.util.internal.logging.Slf4JLogger]
```

可以发现是`Slf4JLogger`。

再查看io.netty.util.internal.logging.Slf4JLogger的实现，发现它内部有一个logger的field：

```
package io.netty.util.internal.logging;

import org.slf4j.Logger;

/**
 * <a href="http://www.slf4j.org/">SLF4J</a> logger.
 */
class Slf4JLogger extends AbstractInternalLogger {
    private static final long serialVersionUID = 108038972685130825L;

    private final transient Logger logger;
```

那么使用arthas的`getstatic`命令来查看这个`logger`属性的值：

```
$ getstatic -c 73ad2d6 io.netty.channel.nio.NioEventLoop logger 'logger'
field: logger
@Logger[
    serialVersionUID=@Long[5454405123156820674],
    FQCN=@String[ch.qos.logback.classic.Logger],
    name=@String[io.netty.channel.nio.NioEventLoop],
    level=null,
    effectiveLevelInt=@Integer[10000],
    parent=@Logger[Logger[io.netty.channel.nio]],
    childrenList=null,
    aai=null,
    additive=@Boolean[true],
    loggerContext=@LoggerContext[ch.qos.logback.classic.LoggerContext[default]],
]
```

可见，logger的最本质实现类是：`ch.qos.logback.classic.Logger`。

再次用`getstatic`命令来确定jar包的location：

```
$ getstatic -c 73ad2d6 io.netty.channel.nio.NioEventLoop logger 'logger.getClass().getProtectionDomain().getCodeSource().getLocation()'
field: logger
@URL[
    BUILTIN_HANDLERS_PREFIX=@String[sun.net.www.protocol],
    serialVersionUID=@Long[-7627629688361524110],
    protocolPathProp=@String[java.protocol.handler.pkgs],
    protocol=@String[jar],
    host=@String[],
    port=@Integer[-1],
    file=@String[file:/opt/app/plugins/tair-plugin/lib/logback-classic-1.2.3.jar!/],
    query=null,
    authority=@String[],
    path=@String[file:/opt/app/plugins/tair-plugin/lib/logback-classic-1.2.3.jar!/],
    userInfo=null,
    ref=null,
    hostAddress=null,
    handler=@Handler[com.taobao.pandora.loader.jar.Handler@1a0c361e],
    hashCode=@Integer[126346621],
    tempState=null,
    factory=@TomcatURLStreamHandlerFactory[org.apache.catalina.webresources.TomcatURLStreamHandlerFactory@3edd7b7],
    handlers=@Hashtable[isEmpty=false;size=4],
    streamHandlerLock=@Object[java.lang.Object@488ccac9],
    serialPersistentFields=@ObjectStreamField[][isEmpty=false;size=7],
]
```

可见这个`ch.qos.logback.classic.Logger`的确是tair插件里加载的。

## 定位logger的level

上面已经定位logger的实现类是`ch.qos.logback.classic.Logger`，但是为什么它会输出`DEBUG` level的日志？

其实在上面的`getstatic -c 73ad2d6 io.netty.channel.nio.NioEventLoop logger 'logger'`输出里，已经打印出它的level是null了。如果对logger有所了解的话，可以知道当child logger的level为null时，它的level取决于parent logger的level。

我们再来看下`ch.qos.logback.classic.Logger`的代码，它有一个parent logger的属性：

```
public final class Logger implements org.slf4j.Logger, LocationAwareLogger, AppenderAttachable<ILoggingEvent>, Serializable {

    /**
     * The parent of this category. All categories have at least one ancestor
     * which is the root category.
     */
    transient private Logger parent;
```

所以，我们可以通过`getstatic`来获取到这个parent属性的内容。然后通过多个parent操作，可以发现level都是null，最终发现ROOT level是DEBUG 。

```
$ getstatic -c 73ad2d6 io.netty.channel.nio.NioEventLoop logger 'logger.parent.parent.parent.parent.parent'
field: logger
@Logger[
    serialVersionUID=@Long[5454405123156820674],
    FQCN=@String[ch.qos.logback.classic.Logger],
    name=@String[ROOT],
    level=@Level[DEBUG],
    effectiveLevelInt=@Integer[10000],
    parent=null,
    childrenList=@CopyOnWriteArrayList[isEmpty=false;size=2],
    aai=@AppenderAttachableImpl[ch.qos.logback.core.spi.AppenderAttachableImpl@1ecf9bae],
    additive=@Boolean[true],
    loggerContext=@LoggerContext[ch.qos.logback.classic.LoggerContext[default]],
]
```

所以 `io.netty.channel.nio.NioEventLoop`的logger的level取的是ROOT logger的配置，即默认值`DEBUG`。

具体实现可以查看`ch.qos.logback.classic.LoggerContext`

```
    public LoggerContext() {
        super();
        this.loggerCache = new ConcurrentHashMap<String, Logger>();

        this.loggerContextRemoteView = new LoggerContextVO(this);
        this.root = new Logger(Logger.ROOT_LOGGER_NAME, null, this);
        this.root.setLevel(Level.DEBUG);
        loggerCache.put(Logger.ROOT_LOGGER_NAME, root);
        initEvaluatorMap();
        size = 1;
        this.frameworkPackages = new ArrayList<String>();
    }
```

## 为什么logback输出到了stdout里

上面我们得到结论

- tair插件里的logback默认的level是DEBUG，导致netty里的日志可以被打印出来

那么我们可以猜测：

- tair里的logback没有特殊配置，或者只配置了tair自己的package，导致ROOT logger的日志直接输出到stdout里

那么实现上tair里是使用了`logger-api`，它通过`LoggerFactory.getLogger`函数获取到了自己package的logger，然后设置level为`INFO`，并设置了appender。

**换而言之，tair插件里的logback没有设置ROOT logger，所以它的默认level是DEBUG，并且默认的appender会输出到stdout里。**

所以tair插件可以增加设置ROOT logger level为`INFO`来修复这个问题。

```
private static com.taobao.middleware.logger.Logger logger
			= com.taobao.middleware.logger.LoggerFactory.getLogger("com.taobao.tair");
	public static com.taobao.middleware.logger.Logger infolog
			= com.taobao.middleware.logger.LoggerFactory.getLogger("com.taobao.tair.custom-infolog");

	public static int JM_LOG_RETAIN_COUNT = 3;
	public static String JM_LOG_FILE_SIZE = "200MB";

	static {
		try {
			String tmp = System.getProperty("JM.LOG.RETAIN.COUNT", "3");
			JM_LOG_RETAIN_COUNT = Integer.parseInt(tmp);
		} catch (NumberFormatException e) {
		}
		JM_LOG_FILE_SIZE = System.getProperty("JM.LOG.FILE.SIZE", "200MB");

		logger.setLevel(Level.INFO);
		logger.activateAppenderWithSizeRolling("tair-client", "tair-client.log", "UTF-8",
				TairUtil.splitSize(JM_LOG_FILE_SIZE, 0.8 / (JM_LOG_RETAIN_COUNT + 1)), JM_LOG_RETAIN_COUNT);
		logger.setAdditivity(false);
		logger.activateAsync(500, 100);

		logger.info("JM_LOG_RETAIN_COUNT " + JM_LOG_RETAIN_COUNT + " JM_LOG_FILE_SIZE " + JM_LOG_FILE_SIZE);

		infolog.setLevel(Level.INFO);
		infolog.activateAppenderWithSizeRolling("tair-client", "tair-client-info.log", "UTF-8", "10MB", 1);
		infolog.setAdditivity(false);
		infolog.activateAsync(500, 100);
```

## 总结

- tair插件里直接以api的方式设置了自己package下的logger
- tair插件里netty的logger的packge和tair并不一样，所以它最终取的是ROOT logger的配置
- logback默认的ROOT logger level是`DEBUG`，输出是stdout
- 利用arthas的`sc`命令定位具体的类
- 利用arthas的`getstatic`获取static filed的值
- 利用logger parent层联的特性，可以向上一层层获取到ROOT logger的配置