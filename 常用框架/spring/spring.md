Spring
=======

#### IOC

 - spring IOC 的实现原理，思考如果自己来实现，如何实现
 
#### spring AOP

1. spring AOP 的实现原理，思考如果自己来实现，如何实现

2. spring AOP和 AspectJ 
  
  很多人认为AspectJ是spring AOP的一部分,这是一个误区，AspectJ是一套独立的面向切面编程的解决方案
  
- [Spring AOP,AspectJ, CGLIB 有点晕](https://www.jianshu.com/p/fe8d1e8bd63e)
- [关于 Spring AOP (AspectJ) 你该知晓的一切](https://blog.csdn.net/javazejian/article/details/56267036)
- [AOP的两种实现-Spring AOP以及AspectJ](https://blog.csdn.net/dm_vincent/article/details/57526325)

#### 事务管理
1. spring 事务管理是如何实现的，实现原理是什么

2. spring 事务传播和几个注解失效的原因
    
    如果理解此问题 一定要理解它的原理
    
        @Service
        class A{
            @Transactinal
            method b(){...}
            
            method a(){    //标记1
                b();
            }
        }
         
        //Spring扫描注解后，创建了另外一个代理类，并为有注解的方法插入一个startTransaction()方法：
        class proxy$A{
            A objectA = new A();
            method b(){    //标记2
                startTransaction();
                objectA.b();
            }
         
            method a(){    //标记3
                objectA.a();    //由于a()没有注解，所以不会启动transaction，而是直接调用A的实例的a()方法
            }
        }
    
- [在同一个类中，一个方法调用另外一个有注解（比如@Async，@Transational）的方法，注解失效的原因和解决方法](https://blog.csdn.net/ClementAD/article/details/47339519)
- [Spring，为内部方法新起一个事务，此处应有坑。](https://www.cnblogs.com/yougewe/p/7466677.html)  （这里面为什么要放到另外一个类里面，可以看上面的一段伪代码就可以理解）

#### spring 原理
- [Spring原理初探----IOC、AOP](https://www.jianshu.com/p/c403609185a5)

#### spring 注解

- [Spring 注解大全与详解](https://blog.csdn.net/weixin_37490221/article/details/78406810)
- [Spring常用注解介绍](https://blog.csdn.net/u010648555/article/details/76299467)

#### spring cache

- [spring cache的使用](https://www.jianshu.com/p/49fc4065201a)
- [Spring Cache抽象详解](http://jinnianshilongnian.iteye.com/blog/2001040/)

#### spring 源码分析

1. 书籍《Spring 源码深度解析》

- [下载地址](https://www.jianshu.com/p/65958f5af34a)

2. github上一个精简实现Spring功能的代码开源项目：

- [tiny-spring](https://github.com/code4craft/tiny-spring)

3. tiny-spring是为了学习Spring的而开发的，可以认为是一个Spring的精简版。Spring的代码很多，层次复杂，阅读起来费劲。我尝试从使用功能的角度出发，参考Spring的实现，一步一步构建，最终完成一个精简版的Spring。有人把程序员与画家做比较，画家有门基本功叫临摹，tiny-spring可以算是一个程序的临摹版本-从自己的需求出发，进行程序设计，同时对著名项目进行参考。

- [Spring拦截器](https://blog.csdn.net/lzy_lizhiyang/article/details/48002645)
- [springboot过滤器和拦截器的实现和区别](https://segmentfault.com/a/1190000012072060)
- [AOP/Spring AOP/Filter/拦截器 释义](https://www.jianshu.com/p/b1e4f7ae45cf)


- [Spring 5 源码解析](http://www.iocoder.cn/Spring/good-collection/)
- [SpringMVC源码总结（一）HandlerMapping和HandlerAdapter入门](http://lgbolgger.iteye.com/blog/2105101)
- [Spring MVC中的HandlerMapping与HandlerAdapter的关系](https://www.cnblogs.com/ASPNET2008/p/6278206.html)
- [Spring 5 源码解析 —— 论 Spring DispatcherServlet 的生命周期](http://www.iocoder.cn/Spring/DispatcherServlet/)
- [springMVC源码分析--HandlerInterceptor拦截器（一）](https://blog.csdn.net/qq924862077/article/details/53524507)  