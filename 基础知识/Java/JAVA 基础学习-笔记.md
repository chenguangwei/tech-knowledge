



[TOC]



## **Java基本数据类型**



[https://github.com/chenguangwei/Java-Tutorial/blob/master/docs/java/basic/2%E3%80%81Java%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md](https://github.com/chenguangwei/Java-Tutorial/blob/master/docs/java/basic/2、Java基本数据类型.md)



变量就是申请内存来存储值。也就是说，当创建变量的时候，需要在内存中申请空间。

内存管理系统根据变量的类型为变量分配存储空间，分配的空间只能用来储存该类型数据。

[![img](https://camo.githubusercontent.com/7f5199036de793bf6f2f530c4105d3a9138b468b/68747470733a2f2f7777772e72756e6f6f622e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031332f31322f6d656d6f7279706963312e6a7067)](https://camo.githubusercontent.com/7f5199036de793bf6f2f530c4105d3a9138b468b/68747470733a2f2f7777772e72756e6f6f622e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031332f31322f6d656d6f7279706963312e6a7067)

因此，通过定义不同类型的变量，可以在内存中储存整数、小数或者字符。

### Java 的两大数据类型:

- 内置数据类型

- 引用数据类型

  #### 内置数据类型

  Java语言提供了八种基本类型。六种数字类型（四个整数型，两个浮点型），一种字符类型，还有一种布尔型。

  ```
  //8位
  byte bx = Byte.MAX_VALUE;
  byte bn = Byte.MIN_VALUE;
  //16位
  short sx = Short.MAX_VALUE;
  short sn = Short.MIN_VALUE;
  //32位
  int ix = Integer.MAX_VALUE;
  int in = Integer.MIN_VALUE;
  //64位
  long lx = Long.MAX_VALUE;
  long ln = Long.MIN_VALUE;
  //32位
  float fx = Float.MAX_VALUE;
  float fn = Float.MIN_VALUE;
  //64位
  double dx = Double.MAX_VALUE;
  double dn = Double.MIN_VALUE;
  //16位
  char cx = Character.MAX_VALUE;
  char cn = Character.MIN_VALUE;
  //1位
  boolean bt = Boolean.TRUE;
  boolean bf = Boolean.FALSE;
  `127`
  `-128`
  `32767`
  `-32768`
  `2147483647`
  `-2147483648`
  `9223372036854775807`
  `-9223372036854775808`
  `3.4028235E38`
  `1.4E-45`
  `1.7976931348623157E308`
  `4.9E-324`
  `�`
  
  `true`
  `false`
  ```

#### 引用类型

- 在Java中，引用类型的变量非常类似于C/C++的指针。引用类型指向一个对象，指向对象的变量是引用变量。这些变量在声明时被指定为一个特定的类型，比如 Employee、Puppy 等。变量一旦声明后，类型就不能被改变了。
- 对象、数组都是引用数据类型。
- 所有引用类型的默认值都是null

 实现总结：其实自动装箱和自动封箱是编译器为我们提供的一颗语法糖。在自动装箱时，编译器调用包装类型的valueOf()方法；在自动拆箱时，编译器调用了相应的xxxValue()方法。



### 自动装箱与拆箱中的“坑”

在使用自动装箱与自动拆箱时，要注意一些陷阱，为了避免这些陷阱，我们有必要去看一下各种包装类型的源码。



1）Integer有一个实例域value，它保存了这个Integer所代表的int型的值，且它是final的，也就是说这个Integer对象一经构造完成，它所代表的值就不能再被改变。

2）Integer重写了equals()方法，它通过比较两个Integer对象的value，来判断是否相等。

3）重点是静态内部类IntegerCache，通过类名就可以发现：它是用来缓存数据的。它有一个数组，里面保存的是连续的Integer对象。 (a) low：代表缓存数据中最小的值，固定是-128。

(b) high：代表缓存数据中最大的值，它可以被该改变，默认是127。high最小是127，最大是Integer.MAX_VALUE-(-low)-1，如果high超过了这个值，那么cache[ ]的长度就超过Integer.MAX_VALUE了，也就溢出了。

(c) cache[]：里面保存着从[low,high]所对应的Integer对象，长度是high-low+1(因为有元素0，所以要加1)。

4）调用valueOf(inti)方法时，首先判断i是否在[low,high]之间，如果是，则复用Integer.cache[i-low]。比如，如果Integer.valueOf(3)，直接返回Integer.cache[131]；如果i不在这个范围，则调用构造方法，构造出一个新的Integer对象。

5）调用intValue()，直接返回value的值。 通过3）和4）可以发现，默认情况下，在使用自动装箱时，VM会复用[-128,127]之间的Integer对象

```
/基本数据类型的常量池是-128到127之间。
// 在这个范围中的基本数据类的包装类可以自动拆箱，比较时直接比较数值大小。
public static void main(String[] args) {
    //int的自动拆箱和装箱只在-128到127范围中进行，超过该范围的两个integer的 == 判断是会返回false的。
    Integer a1 = 128;
    Integer a2 = -128;
    Integer a3 = -128;
    Integer a4 = 128;
    System.out.println(a1 == a4);
    System.out.println(a2 == a3);

    Byte b1 = 127;
    Byte b2 = 127;
    Byte b3 = -128;
    Byte b4 = -128;
    //byte都是相等的，因为范围就在-128到127之间
    System.out.println(b1 == b2);
    System.out.println(b3 == b4);

    //
    Long c1 = 128L;
    Long c2 = 128L;
    Long c3 = -128L;
    Long c4 = -128L;
    System.out.println(c1 == c2);
    System.out.println(c3 == c4);

    //char没有负值
    //发现char也是在0到127之间自动拆箱
    Character d1 = 128;
    Character d2 = 128;
    Character d3 = 127;
    Character d4 = 127;
    System.out.println(d1 == d2);
    System.out.println(d3 == d4);


    `结果`
    
    `false`
    `true`
    `true`
    `true`
    `false`
    `true`
    `false`
    `true`
    
    

    Integer i = 10;
    Byte b = 10;
    //比较Byte和Integer.两个对象无法直接比较，报错
    //System.out.println(i == b);
    System.out.println("i == b " + i.equals(b));
    //答案是false,因为包装类的比较时先比较是否是同一个类，不是的话直接返回false.
    int ii = 128;
    short ss = 128;
    long ll = 128;
    char cc = 128;
    System.out.println("ii == bb " + (ii == ss));
    System.out.println("ii == ll " + (ii == ll));
    System.out.println("ii == cc " + (ii == cc));
    
    结果
    i == b false
    ii == bb true
    ii == ll true
    ii == cc true
    
    //这时候都是true，因为基本数据类型直接比较值，值一样就可以
    
    
```

##### 总结：

通过上面的代码，我们分析一下自动装箱与拆箱发生的时机：

（1）当需要一个对象的时候会自动装箱，比如Integer a = 10;equals(Object o)方法的参数是Object对象，所以需要装箱。

（2）当需要一个基本类型时会自动拆箱，比如int a = new Integer(10);算术运算是在基本类型间进行的，所以当遇到算术运算时会自动拆箱，比如代码中的 c == (a + b);

（3） 包装类型 == 基本类型时，包装类型自动拆箱；

需要注意的是：“==”在没遇到算术运算时，不会自动拆箱；基本类型只会自动装箱为对应的包装类型，代码中最后一条说明的内容。

在JDK 1.5中提供了自动装箱与自动拆箱，这其实是Java 编译器的语法糖，编译器通过调用包装类型的valueOf()方法实现自动装箱，调用xxxValue()方法自动拆箱。自动装箱和拆箱会有一些陷阱，那就是包装类型复用了某些对象。

（1）Integer默认复用了[-128,127]这些对象，其中高位置可以修改；

（2）Byte复用了全部256个对象[-128,127]；

（3）Short复用了[-128,127]这些对象；

（4）Long复用了[-128,127];

（5）Character复用了[0,127],Charater不能表示负数;

Double和Float是连续不可数的，所以没法复用对象，也就不存在自动装箱复用陷阱。

Boolean没有自动装箱与拆箱，它也复用了Boolean.TRUE和Boolean.FALSE，通过Boolean.valueOf(boolean b)返回的Blooean对象要么是TRUE，要么是FALSE，这点也要注意。



### String 和包装类



而String类型可以通过intern来完成这个操作。

JDK1.7后，常量池被放入到堆空间中，这导致intern()函数的功能不同，具体怎么个不同法，且看看下面代码，这个例子是网上流传较广的一个例子，分析图也是直接粘贴过来的，这里我会用自己的理解去解释这个例子：

```
[java] view plain copy
String s = new String("1");  
s.intern();  
String s2 = "1";  
System.out.println(s == s2);  
  
String s3 = new String("1") + new String("1");  
s3.intern();  
String s4 = "11";  
System.out.println(s3 == s4);  
输出结果为：

[java] view plain copy
JDK1.6以及以下：false false  
JDK1.7以及以上：false true
```

[![image](https://camo.githubusercontent.com/3daeaf6669ebc6e17e3869f3e87ef7d45237f6a9/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f32303138303432323233313931363738383f77617465726d61726b2f322f746578742f6148523063484d364c7939696247396e4c6d4e7a5a473475626d56304c3245334d6a51344f44673d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f3730)](https://camo.githubusercontent.com/3daeaf6669ebc6e17e3869f3e87ef7d45237f6a9/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f32303138303432323233313931363738383f77617465726d61726b2f322f746578742f6148523063484d364c7939696247396e4c6d4e7a5a473475626d56304c3245334d6a51344f44673d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f3730)

[![image](https://camo.githubusercontent.com/7e0bdac06bd65e717e63a8580057a565e1ff2047/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f32303138303432323233313932393431333f77617465726d61726b2f322f746578742f6148523063484d364c7939696247396e4c6d4e7a5a473475626d56304c3245334d6a51344f44673d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f3730)](https://camo.githubusercontent.com/7e0bdac06bd65e717e63a8580057a565e1ff2047/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f32303138303432323233313932393431333f77617465726d61726b2f322f746578742f6148523063484d364c7939696247396e4c6d4e7a5a473475626d56304c3245334d6a51344f44673d2f666f6e742f3561364c354c32542f666f6e7473697a652f3430302f66696c6c2f49304a42516b46434d413d3d2f646973736f6c76652f3730) JDK1.6查找到常量池存在相同值的对象时会直接返回该对象的地址。

JDK 1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前没有区别，区别在于，如果在常量池找不到对应的字符串，则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串的引用。

那么其他字符串在常量池找值时就会返回另一个堆中对象的地址。





### final使用

#### final关键字的知识点

1. final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。final变量一旦被初始化后不能再次赋值。
2. 本地变量必须在声明时赋值。 因为没有初始化的过程
3. 在匿名类中所有变量都必须是final变量。
4. final方法不能被重写, final类不能被继承
5. 接口中声明的所有变量本身是final的。类似于匿名类
6. final和abstract这两个关键字是反相关的，final类就不可能是abstract的。
7. final方法在编译阶段绑定，称为静态绑定(static binding)。
8. 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化。

final方法的好处:

1. 提高了性能，JVM在常量池中会缓存final变量
2. final变量在多线程中并发安全，无需额外的同步开销
3. final方法是静态编译的，提高了调用速度
4. **final类创建的对象是只可读的，在多线程可以安全共享**