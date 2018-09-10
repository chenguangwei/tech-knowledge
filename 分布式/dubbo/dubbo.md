dubbo
=========

> dubbo 原理


> dubbo中的负载均衡和容错机制


> dubbo如果自己实现


> dubbo和JDK版本要对应

> dubbo 解析
> - [dubbo解析文档](https://dubbo.gitbooks.io/dubbo-user-book/preface/background.html)


> dubbo FAQ

1. Dubbo的广播模式下Can't assign requested address问题
   解决方法：造成这种原因的主要是系统中开启了IPV6协议，java网络编程经常会获取到IPv6的地址。解决方法：
        添加vm参数 -Djava.net.preferIPv4Stack=true
        
dubbo常见问题和解决方案

- [使用dubbo过程中遇到的问题](https://blog.csdn.net/u012100371/article/details/78849813)