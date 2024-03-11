java8
======
- [Java 8的新特性—终极版](https://www.jianshu.com/p/5b800057f2d8)
- [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)
- [java8——并行处理与性能](https://www.jianshu.com/p/23a013bb0d81)
- [Java8并行流使用注意事项](https://www.cnblogs.com/gaobig/p/4874400.html)
- [Java 8 示例](https://blog.godiscoder.cn/%E5%8F%AA%E8%B0%88%E9%A3%8E%E6%9C%88/article/5b1e6447e0cec5435c2be970)

1、 并发性

基于新增的lambda表达式和steam特性，为Java 8中为java.util.concurrent.ConcurrentHashMap类添加了新的方法来支持聚焦操作；另外，也为java.util.concurrentForkJoinPool类添加了新的方法来支持通用线程池操作（更多内容可以参考我们的并发编程课程）。
Java 8还添加了新的java.util.concurrent.locks.StampedLock类，用于支持基于容量的锁——该锁有三个模型用于支持读写操作（可以把这个锁当做是java.util.concurrent.locks.ReadWriteLock的替代者）。
在java.util.concurrent.atomic包中也新增了不少工具类，列举如下：

DoubleAccumulator
DoubleAdder
LongAccumulator
LongAdder





2、 JVM的新特性
使用Metaspace（JEP 122）代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize。

- [Java8内存模型—永久代(PermGen)和元空间(Metaspace)](https://www.cnblogs.com/paddix/p/5309550.html)
- [JDK8 的FullGC 之 metaspace](https://www.jianshu.com/p/1a0b4bf8d498)
- [JVM(二)Java8内存划分](https://blog.csdn.net/yjp198713/article/details/78759933)


3、Supplier 接口

