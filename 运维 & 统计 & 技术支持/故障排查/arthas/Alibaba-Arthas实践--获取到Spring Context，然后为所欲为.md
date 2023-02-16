# Alibaba Arthas实践--获取到Spring Context，然后为所欲为

## 背景

Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。

- https://github.com/alibaba/arthas

Arthas提供了非常丰富的关于调用拦截的命令，比如 trace/watch/monitor/tt 。但是很多时候我们在排查问题时，需要更多的线索，并不只是函数的参数和返回值。
比如在一个spring应用里，想获取到spring context里的其它bean。如果能随意获取到spring bean，那就可以“为所欲为”了。

下面介绍如何利用Arthas获取到spring context。

Demo： https://github.com/hengyunabc/spring-boot-inside/tree/master/demo-arthas-spring-boot

Arthas快速开始：https://alibaba.github.io/arthas/quick-start.html

## 使用tt命令获取到spring context

Demo是一个spring mvc应用，请求会经过一系列的spring bean处理，那么我们可以在spring mvc的类里拦截到一些请求。

启动Demo： `mvn spring-boot:run`

使用Arthas Attach成功之后，执行`tt`命令来记录`RequestMappingHandlerAdapter#invokeHandlerMethod`的请求

```
tt -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod
```

然后访问一个网页： http://localhost:8080/

可以看到Arthas会拦截到这个调用，index是1000，并且打印出：

```
$ watch com.example.demo.Test * 'params[0]@sss'
$ tt -t org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter invokeHandlerMethod
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 101 ms.
 INDEX  TIMESTAMP         COST(ms  IS-RE  IS-EX  OBJECT       CLASS                     METHOD
                          )        T      P
------------------------------------------------------------------------------------------------------------------
 1000   2019-01-27 16:31  3.66744  true   false  0x4465cf70   RequestMappingHandlerAda  invokeHandlerMethod
        :54                                                   pter
```

**那么怎样获取到spring context？**

可以用`tt`命令的`-i`参数来指定index，并且用`-w`参数来执行ognl表达式来获取spring context：

```
$ tt -i 1000 -w 'target.getApplicationContext()'
@AnnotationConfigEmbeddedWebApplicationContext[
    reader=@AnnotatedBeanDefinitionReader[org.springframework.context.annotation.AnnotatedBeanDefinitionReader@35dc90ec],
    scanner=@ClassPathBeanDefinitionScanner[org.springframework.context.annotation.ClassPathBeanDefinitionScanner@72078a14],
    annotatedClasses=null,
    basePackages=null,
]
Affect(row-cnt:1) cost in 7 ms.
```

## 从spring context里获取任意bean

获取到spring context之后，就可以获取到任意的bean了，比如获取到`helloWorldService`，并调用`getHelloMessage()`函数：

```
$ tt -i 1000 -w 'target.getApplicationContext().getBean("helloWorldService").getHelloMessage()'
@String[Hello World]
Affect(row-cnt:1) cost in 5 ms.
```

## 更多的思路

在很多代码里都有static函数或者static holder类，顺滕摸瓜，可以获取很多其它的对象。比如在Dubbo里通过`SpringExtensionFactory`获取spring context：

```
$ ognl '#context=@com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory@contexts.iterator.next, 
#context.getBean("userServiceImpl").findUser(1)'
@User[
    id=@Integer[1],
    name=@String[Deanna Borer],
]
```

## 链接

- Arthas: https://github.com/alibaba/arthas
- https://alibaba.github.io/arthas/tt.html
- https://alibaba.github.io/arthas/ognl.html