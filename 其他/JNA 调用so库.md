JNA调用so库



使用java的jna调用c++的so文件
整体描述
前期准备

1. 上传so文件
2. 修改配置文件
   java端代码
3. 引入jni依赖
   2.创建jna类
   3.调用so文件中的方法
   排坑指南
   问题1
   问题2
   整体描述
   最近项目需要调用so文件，c++的同事给了一个so文件，在java端使用jna调用，记录一下具体操作和遇到的坑…本文的配置方法都是在linux下的配置，因为最后程序也是在linux下运行。

前期准备

1. 上传so文件
   将so文件上传到java的lib目录下，一般会在/etc/profile文件里配置java的lib目录

2. 修改配置文件
   如果这个so文件可能还要依赖其他的so文件，这个就要问下写so文件的人，看看他们是在哪个路径读取的so文件，我的项目里修改的是/etc/ld.so.conf文件，将so中依赖的其他so文件路径加上，修改之后使用ldconfig命令更新配置文件(这一步可以先跳过，如果调用so失败时再看)

java端代码

1. 引入jni依赖
   
        <dependency>
            <groupId>net.java.dev.jna</groupId>
            <artifactId>jna-platform</artifactId>
        </dependency>
        <dependency>
            <groupId>net.java.dev.jna</groupId>
            <artifactId>jna</artifactId>
            <version>4.4.0</version>
        </dependency>
   
   
   2.创建jna类
   此处需要注意的是：so文件名称为libtest.so，在代码中写so文件名称时，写test就可以

package com.thcb.jna;

import com.sun.jna.Library;
import com.sun.jna.Native;

/**

* @author thcb
  */

public interface JnaTest extends Library {

    /**
     * jna库，test为so文件名称
     */
    public static final Test jnaTest = (JnaTest) Native.loadLibrary("jnatest", JnaTest.class);
    
    /**
     * 调用so文件，jna_test为so文件中的方法
     *
     * @param username        登陆用户名
     * @param passwd          登陆密码
     * @return
     */
    public abstract int jna_test(String username, String passwd) throws IllegalArgumentException;

3.调用so文件中的方法
      try {
             int result = JnaTest.jnaTest.jna_test(account, pwd);
             log.info("result:{}", result);
      } catch (Throwable t) {
           log.error("Exception:{}", t.toString());
      }


try catch很有必要，可以catch一些so文件调用的异常，避免程序崩溃，但是so文件中的方法抛出的异常在java是没法catch的，这块只能让c++开发时自己捕捉异常。

排坑指南
按照如上步骤，就可以实现java调用c++的so文件了，如果一切顺利没有报错，那么恭喜你后面的内容就不用看了(能一次就过，谢天谢地)

问题1
Exception:java.lang.UnsatisfiedLinkError: Unable to load library ‘jnatest’: Native library (darwin/libjnatest.dylib) not found in resource path
错误说明：意思就是so文件没找到
解决办法：这时候就要看看上面前期准备2提到的问题，还有so文件路径，java的lib路径的问题，java的路径一般配置在/etc/profile文件中

问题2
java: symbol lookup error: /app/data/packet/lib/librelayvoice.so: undefined symbol：XXXXXX
错误说明：后面XXXXX是找不到的方法，该错误就是没有找到so文件中的XXXXXX符号，一般来说是c++的函数名等。这个错误直接把java虚拟机jvm干崩了…
解决办法：一般情况下，这个问题就是so文件的问题了，需要重新编译so文件，可以用指令查看下so文件中使用其他的so：

ldd 【so文件名】

看看so文件中包含了哪些，我的问题就是我用的so文件中引用了其他的so文件里面的方法，但是引用的这个so文件并没有链接到我用的so文件中，导致报错。我找c++的同事要了cpp源码和.h文件，然后自己编译一下：

g++  -shared  -fPIC -L./lib - test234.so -o libtest.so test.cpp

通过cpp生成so文件，test234.so就是我的so需要引用的其他so文件的名称，前面的 -L./lib，/lib就是test234.so的目录，后面的libtest.so是生成so文件的名称，test.cpp就是cpp源文件。

使用新编译的so文件替换之前的so文件，需要重新启动一下java程序，就可以了，说明就是这个问题。

