# Arthas实践：是哪个Controller处理了请求？



## 背景

Arthas是阿里巴巴开源的Java诊断利器，深受开发者喜爱。

- https://github.com/alibaba/arthas
- [Arthas在线教程](https://alibaba.github.io/arthas/arthas-tutorials?language=cn)

之前分享了Arthas怎样排查 404/401 的问题: http://hengyunabc.github.io/arthas-spring-boot-404-401/

我们可以快速定位一个请求是被哪些`Filter`拦截的，或者请求最终是由哪些`Servlet`处理的。

但有时，我们想知道一个请求是被哪个Spring MVC Controller处理的。如果翻代码的话，会比较难找，并且不一定准确。

通过Arthas可以精确定位是哪个`Controller`处理请求。

## Demo

还是以这个demo为例： https://github.com/hengyunabc/spring-boot-inside/tree/master/demo-404-401

启动之后，访问： http://localhost:8080/user/1 ，会返回一个user对象。那么这个请求是被哪个`Controller`处理的呢？

## trace定位DispatcherServlet

我们先试下跟踪`Servlet`：

```
trace javax.servlet.Servlet *
```

从trace的结果可以看出来，请求最终是被`DispatcherServlet#doDispatch()`处理了，但是没有办法知道是哪个`Controller`处理。

```
`---[27.453122ms] org.springframework.web.servlet.DispatcherServlet:doDispatch()
    +---[0.005822ms] org.springframework.web.context.request.async.WebAsyncUtils:getAsyncManager() #929
    +---[0.107365ms] org.springframework.web.servlet.DispatcherServlet:checkMultipart() #936
    |   `---[0.062451ms] org.springframework.web.servlet.DispatcherServlet:checkMultipart()
    |       `---[0.016924ms] org.springframework.web.multipart.MultipartResolver:isMultipart() #1093
    +---[2.103935ms] org.springframework.web.servlet.DispatcherServlet:getHandler() #940
    |   `---[2.036042ms] org.springframework.web.servlet.DispatcherServlet:getHandler()
```

## watch定位handler

trace结果里把调用的行号打印出来了，我们可以直接在IDE里查看代码（也可以用jad命令反编译）：

```
// org.springframework.web.servlet.DispatcherServlet
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
```

- 仔细看代码，可以发现`mappedHandler = getHandler(processedRequest);`得到了处理请求的handler

下面用`watch`命令来获取`getHandler`函数的返回结果。

`watch`之后，再次访问 http://localhost:8080/user/1

```
$ watch org.springframework.web.servlet.DispatcherServlet getHandler returnObj
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 332 ms.
ts=2019-06-04 11:38:06; [cost=2.75218ms] result=@HandlerExecutionChain[
    logger=@SLF4JLocationAwareLog[org.apache.commons.logging.impl.SLF4JLocationAwareLog@665c08a],
    handler=@HandlerMethod[public com.example.demo.arthas.user.User com.example.demo.arthas.user.UserController.findUserById(java.lang.Integer)],
    interceptors=null,
    interceptorList=@ArrayList[isEmpty=false;size=2],
    interceptorIndex=@Integer[-1],
]
```

可以看到处理请求的handler是 `om.example.demo.arthas.user.UserController.findUserById`。

## 总结

- Spring MVC的请求是在`DispatcherServlet`分发，查找到对应的`mappedHandler`来处理
- 使用Arthas时，灵活结合代码，可以快速精确定位问题

## 链接

- https://github.com/alibaba/arthas
- https://alibaba.github.io/arthas/watch.html
- https://alibaba.github.io/arthas/trace.html

## 原文链接

- http://hengyunabc.github.io/arthas-springmvc-controller/