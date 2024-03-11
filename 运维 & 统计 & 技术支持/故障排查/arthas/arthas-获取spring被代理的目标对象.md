# arthas 获取spring被代理的目标对象



## 背景

记得一次问题排查，通过ognl 获取到 spring aop 代理过的cglib 代理对象的原始对象获取问题，spring的静态static spring context 进行调用获取被代理的目标对象的问题，记得当事是通过内部的一个工具 [代理对象中被代理的目标对象](https://www.letianbiji.com/spring/spring-get-target-from-proxy.html) 类似这个方法，通过静态的方法进行调用.挺方便的，但是这个方法比较麻烦，不是所有的工程都有这个方法，如何通过工具化让大家都能使用，这里使用 ognl 表达式进行复原整个过程，方便使用。[更多使用参考 Idea Plugin](https://arthas.aliyun.com/doc/idea-plugin.html),最近会把这个功能集成工具化，方便使用。

### 获取目标bean的命令

```
ognl -x 3 '#springContext=@com.xxx.util.SpringUtil@context,#springContext.getBean("BeanName")' -c 5600a5da
```

### 获取被代理的目标对象命令

```
ognl -x 3 '#springContext=@com.xxx.util.SpringUtil@context,#proxyBean = #springContext.getBean("BeanName"),@AopTargetUtils@getTarget(#proxyBean)' -c 5600a5da
```

## 参考文章

### Ongl Lambda表达式

[Ongl 官方文档](https://commons.apache.org/proper/commons-ognl/language-guide.html#Pseudo-Lambda Expressions)
定义了一个Ongl Lambda表达式, Lambda表达式必须放在方括号内部，#this表示表达式的参数

```
#fib =:[#this==0 ? 0 : #this==1 ? 1 : #fib(#this-2)+#fib(#this-1)]

#fin(11)  //表示调用这个函数 计算 斐布拉基数列
```

### Spring 获取代理对象中被代理的目标对象

[代理对象中被代理的目标对象](https://www.letianbiji.com/spring/spring-get-target-from-proxy.html)

```
import java.lang.reflect.Field;  

import org.springframework.aop.framework.AdvisedSupport;  
import org.springframework.aop.framework.AopProxy;  
import org.springframework.aop.support.AopUtils;  

public class AopTargetUtils {  


    /** 
     * 获取 目标对象 
     * @param proxy 代理对象 
     * @return  
     * @throws Exception 
     */  
    public static Object getTarget(Object proxy) throws Exception {  

        if(!AopUtils.isAopProxy(proxy)) {  
            return proxy; //不是代理对象  
        }  

        if(AopUtils.isJdkDynamicProxy(proxy)) {  
            return getJdkDynamicProxyTargetObject(proxy);  
        } else { //cglib  
            return getCglibProxyTargetObject(proxy);  
        }  

    }  


    private static Object getCglibProxyTargetObject(Object proxy) throws Exception {  
        Field h = proxy.getClass().getDeclaredField("CGLIB$CALLBACK_0");  
        h.setAccessible(true);  
        Object dynamicAdvisedInterceptor = h.get(proxy);  

        Field advised = dynamicAdvisedInterceptor.getClass().getDeclaredField("advised");  
        advised.setAccessible(true);  

        Object target = ((AdvisedSupport)advised.get(dynamicAdvisedInterceptor)).getTargetSource().getTarget();  

        return target;  
    }  

    private static Object getJdkDynamicProxyTargetObject(Object proxy) throws Exception {  
        Field h = proxy.getClass().getSuperclass().getDeclaredField("h");  
        h.setAccessible(true);  
        AopProxy aopProxy = (AopProxy) h.get(proxy);  

        Field advised = aopProxy.getClass().getDeclaredField("advised");  
        advised.setAccessible(true);  

        Object target = ((AdvisedSupport)advised.get(aopProxy)).getTargetSource().getTarget();  

        return target;  
    }  

}
```

## 实践获取目标对象

tt -w 执行本身使用ognl 去去处理的,类似的还有watch trace 等等

### 获取spring cglib的aop原始对象

```
tt -w '#cglibTarget =:[#hField =#this.getClass().getDeclaredField("CGLIB$CALLBACK_0"),#hField.setAccessible(true),#dynamicAdvisedInterceptor=#hField.get(#this),#fieldAdvised=#dynamicAdvisedInterceptor.getClass().getDeclaredField("advised"),#fieldAdvised.setAccessible(true),1==1? #fieldAdvised.get(#dynamicAdvisedInterceptor).getTargetSource().getTarget():null],#cglibTarget(target)'  -x 1 -i 1001
```

### 获取spring aop jdk的原始对象

```
tt -w '#jdkTarget=:[ #hField=#this.getClass().getSuperclass().getDeclaredField("h"),#hField.setAccessible(true),#aopProxy=#hField.get(#this),#advisedField=#aopProxy.getClass().getDeclaredField("advised"),#advisedField.setAccessible(true),1==1?#advisedField.get(#aopProxy).getTargetSource().getTarget():null],#jdkTarget(target)'  -x 1 -i 1001
```

### 当前是否为spring的Aop代理对象

```
tt -w '#isProxy=:[ @org.springframework.aop.support.AopUtils@isAopProxy(#this)?true:false],#isProxy(target)'  -x 1 -i 1001
```

### 当前是否为spring jdk的代理对象

```
tt -w '#isJdkDynamicProxy =:[@org.springframework.aop.support.AopUtils@isJdkDynamicProxy(#this) ? true :false ],#isJdkDynamicProxy(target)'  -x 1 -i 1001
```

### 结合上面的一起,获取spring 代理对象的target

```
tt -w '#isProxy=:[ @org.springframework.aop.support.AopUtils@isAopProxy(#this)?true:false],#isJdkDynamicProxy =:[@org.springframework.aop.support.AopUtils@isJdkDynamicProxy(#this) ? true :false ],#cglibTarget =:[#hField =#this.getClass().getDeclaredField("CGLIB$CALLBACK_0"),#hField.setAccessible(true),#dynamicAdvisedInterceptor=#hField.get(#this),#fieldAdvised=#dynamicAdvisedInterceptor.getClass().getDeclaredField("advised"),#fieldAdvised.setAccessible(true),1==1? #fieldAdvised.get(#dynamicAdvisedInterceptor).getTargetSource().getTarget():null],#jdkTarget=:[ #hField=#this.getClass().getSuperclass().getDeclaredField("h"),#hField.setAccessible(true),#aopProxy=#hField.get(#this),#advisedField=#aopProxy.getClass().getDeclaredField("advised"),#advisedField.setAccessible(true),1==1?#advisedField.get(#aopProxy).getTargetSource().getTarget():null],#nonProxyResultFunc = :[!#isProxy(#this) ? #this :#isJdkDynamicProxy(#this)? #isJdkDynamicProxy(#this) : #cglibTarget(#this)],#nonProxyTarget=#nonProxyResultFunc(target),#nonProxyTarget'  -x 1 -i 1001
```

### 纯JDK 动态代理 获取InvocationHandler

```
tt -w '#isProxy=:[ #this instanceof java.lang.reflect.Proxy ? @java.lang.reflect.Proxy@getInvocationHandler(#this) : #this],#isProxy(target)'  -x 1 -i 1002
```

### 结合上面所有的 如果是spring 代理对象 获取原始的对象，如果是jdk 动态代理获取InvocationHandler 否则返回target本身

```
tt -w '#isProxy=:[ @org.springframework.aop.support.AopUtils@isAopProxy(#this)?1: #this instanceof java.lang.reflect.Proxy ? 0 :-1],#isJdkDynamicProxy =:[@org.springframework.aop.support.AopUtils@isJdkDynamicProxy(#this) ? true :false ],#cglibTarget =:[#hField =#this.getClass().getDeclaredField("CGLIB$CALLBACK_0"),#hField.setAccessible(true),#dynamicAdvisedInterceptor=#hField.get(#this),#fieldAdvised=#dynamicAdvisedInterceptor.getClass().getDeclaredField("advised"),#fieldAdvised.setAccessible(true),1==1? #fieldAdvised.get(#dynamicAdvisedInterceptor).getTargetSource().getTarget():null],#jdkTarget=:[ #hField=#this.getClass().getSuperclass().getDeclaredField("h"),#hField.setAccessible(true),#aopProxy=#hField.get(#this),#advisedField=#aopProxy.getClass().getDeclaredField("advised"),#advisedField.setAccessible(true),1==1?#advisedField.get(#aopProxy).getTargetSource().getTarget():null],#nonProxyResultFunc = :[#proxyResul=#isProxy(#this),#proxyResul== -1 ?#this :#proxyResul== 0? @java.lang.reflect.Proxy@getInvocationHandler(#this):#isJdkDynamicProxy(#this)? #isJdkDynamicProxy(#this) : #cglibTarget(#this)],#nonProxyTarget=#nonProxyResultFunc(target),#nonProxyTarget'  -x 1 -i 1002
```

## 总结

ognl 表达式学习起来很奇妙, sc -d xxxClass 获取所有的当前类相关的类 ，然后通过jad 反编译查看 源码，这样理解一些开源框架非常方便。