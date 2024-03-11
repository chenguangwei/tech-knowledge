# Arthas实践--使用redefine排查应用奇怪的日志来源



## 背景

随着应用越来越复杂，依赖越来越多，日志系统越来越混乱，有时会出现一些奇怪的日志，比如：

```
[] [] [] No credential found
```

那么怎样排查这些奇怪的日志从哪里打印出来的呢？因为搞不清楚是什么logger打印出来的，所以想定位就比较头疼。

下面介绍用arthas的redefine命令快速定位奇怪日志来源。

- Arthas: https://github.com/alibaba/arthas
- redefine命令：https://alibaba.github.io/arthas/redefine.html

## 修改StringBuilder

首先在java代码里，字符串拼接基本都是通过`StringBuilder`来实现的。比如下面的代码：

```
	public static String hello(String world) {
		return "hello " + world;
	}
```

实际上生成的字节码也是用`StringBuilder`来拼接的：

```
  public static java.lang.String hello(java.lang.String);
    descriptor: (Ljava/lang/String;)Ljava/lang/String;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=1, args_size=1
         0: new           #22                 // class java/lang/StringBuilder
         3: dup
         4: ldc           #24                 // String hello
         6: invokespecial #26                 // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
         9: aload_0
        10: invokevirtual #29                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        13: invokevirtual #33                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        16: areturn
```

在java的logger系统里，输出日志时通常也是`StringBuilder`来实现的，最终会调用`StringBuilder.toString()`，那么我们可以修改`StringBuilder`的代码来检测到日志来源。

`StringBuilder.toString()` 的原生实现是：

```
    @Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
```

修改为：

```
    @Override
    public String toString() {
        // Create a copy, don't share the array
    	String result = new String(value, 0, count);
    	if(result.contains("No credential found")) {
    		System.err.println(result);
    		new Throwable().printStackTrace();
    	}
        return result;
    }
```

增加的逻辑是：**当String里包含`No credential found`时打印出当前栈，这样子就可以定位日志输出来源了。**

## 编绎修改过的StringBuilder

其实很简单，在IDE里把`StringBuilder`的代码复制一份，然后贴到任意一个工程里，然后编绎即可。

也可以直接用javac来编绎：

```
javac StringBuilder.java
```

## 启动应用，使用Arthas redefine修改过的StringBuilder

启动应用后，在奇怪日志输出之前，先使用arthas attach应用，再redefine StringBuilder:

```
$ redefine  /tmp/StringBuilder.class
redefine success, size: 1
```

当执行到输出`[] [] [] No credential found`的logger代码时，会打印当前栈。实际运行结果是：

```
[] [] [] No credential found
java.lang.Throwable
	at java.lang.StringBuilder.toString(StringBuilder.java:410)
	at com.taobao.middleware.logger.util.MessageUtil.getMessage(MessageUtil.java:26)
	at com.taobao.middleware.logger.util.MessageUtil.getMessage(MessageUtil.java:15)
	at com.taobao.middleware.logger.slf4j.Slf4jLogger.info(Slf4jLogger.java:77)
	at com.taobao.spas.sdk.common.log.SpasLogger.info(SpasLogger.java:18)
	at com.taobao.spas.sdk.client.identity.CredentialWatcher.loadCredential(CredentialWatcher.java:128)
	at com.taobao.spas.sdk.client.identity.CredentialWatcher.access$200(CredentialWatcher.java:18)
	at com.taobao.spas.sdk.client.identity.CredentialWatcher$1.run(CredentialWatcher.java:58)
	at java.util.TimerThread.mainLoop(Timer.java:555)
	at java.util.TimerThread.run(Timer.java:505)
```

**可以看到是`spas.sdk`打印出了`[] [] [] No credential found`的日志。**

## 总结

- logger最终会用StringBuilder来输出
- 修改StringBuilder来定位输出特定日志的地方
- 使用Arthas redefine命令来加载修改过的StringBuilder
- redefine命令实际上实现了任意代码线上debug的功能，可以随意本地修改代码重新编绎，然后线上redefine加载
- redefine的功能过于强大，所以请小心使用:)