MapReduce
=====

MapReduce是一种*编程模型*，用于大规模数据集（大于1TB）的并行运算。概念"Map（_映射_）"和"Reduce（_归约_）"，是它们的主要思想，都是从函数式编程语言里借来的，还有从矢量编程语言里借来的特性。它极大地方便了编程人员在不会分布式并行编程的情况下，将自己的程序运行在分布式系统上。 当前的软件实现是指定一个Map（映射）函数，用来把一组键值对映射成一组新的键值对，指定并发的Reduce（归约）函数，用来保证所有映射的键值对中的每一个共享相同的键组

- [MapReduce](https://baike.baidu.com/item/MapReduce)
- [关于MapReduce的理解？](https://www.zhihu.com/question/23345991)