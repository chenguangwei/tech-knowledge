dubbo
=========

> dubbo 原理


> dubbo中的负载均衡和容错机制


> dubbo如果自己实现


> dubbo和JDK版本要对应

> dubbo 解析
> - [dubbo用户解析文档](https://dubbo.gitbooks.io/dubbo-user-book/preface/background.html)


> dubbo FAQ

1. Dubbo的广播模式下Can't assign requested address问题
   解决方法：造成这种原因的主要是系统中开启了IPV6协议，java网络编程经常会获取到IPv6的地址。解决方法：
        添加vm参数 -Djava.net.preferIPv4Stack=true
        
dubbo常见问题和解决方案

- [使用dubbo过程中遇到的问题](https://blog.csdn.net/u012100371/article/details/78849813)

dubbo异常处理/传递原理

- [基于dubbo的分布式应用中的统一异常处理](http://xurui.pro/2018/05/29/%E5%9F%BA%E4%BA%8Edubbo%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E5%BA%94%E7%94%A8%E4%B8%AD%E7%9A%84%E7%BB%9F%E4%B8%80%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)
- [dubbo(四)异常处理](https://blog.csdn.net/qq315737546/article/details/53915067)