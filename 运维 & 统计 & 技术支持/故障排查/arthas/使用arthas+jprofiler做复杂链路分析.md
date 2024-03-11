# 使用arthas+jprofiler做复杂链路分析



## 背景

arthas提供了profiler命令，可以生成热点火焰图。通过采样录制调用链路来做性能分析，极大提升了线上排查性能问题的效率。

但是有一个问题，当async-profiler全量采样导出的svg文件太大时，想要找到关键的调用点，就非常困难。

比如下图：
[![1](https://user-images.githubusercontent.com/716461/89874894-1093a680-dbef-11ea-8c58-b6d56e6ecb3b.jpg)](https://user-images.githubusercontent.com/716461/89874894-1093a680-dbef-11ea-8c58-b6d56e6ecb3b.jpg)

没有办法做聚合或过滤，这方面本地的profiler工具比如jprofiler、yourkits就方便很多，有没有办法将两者结合起来呢？

经过分析发现，async-profiler支持jfr (Java Flight Recorder)格式输出，jprofiler也支持打开jfr快照，成了！具体操作步骤如下：

### 1. arthas采样生成jfr文件

启动arthas之后，执行以下采样命令：

```
profiler start -f /home/admin/yourAppName/target/arthas-output/%t.jfr -d 180
```

%t 表示当前时间，-d 后面是采样秒数，更多参数参见：
https://alibaba.github.io/arthas/profiler.html
https://github.com/jvm-profiling-tools/async-profiler/blob/v1.6/src/arguments.cpp

### 2. 下载jfr到本地

可以用oss倒腾，或者szrz等其他途径倒腾到本地。

### 3. jprofiler分析

在做性能分析时我们常常想要找出：是谁在调用我，是谁调用我最多。下面举例介绍怎么做的。

#### 3.1 打开快照

使用jprofiler打开jfr文件，选择Open a snapshot, 打开之后选择CPU views

[![2](https://user-images.githubusercontent.com/716461/89875351-bb0bc980-dbef-11ea-8092-b359f42459fa.jpg)](https://user-images.githubusercontent.com/716461/89875351-bb0bc980-dbef-11ea-8092-b359f42459fa.jpg)

#### 3.2 反向分析

View -> Find 查找要分析的类和方法，然后选择 Analyze -> Calculate Backtraces to Selected Method

[![3](https://user-images.githubusercontent.com/716461/89875294-acbdad80-dbef-11ea-88bb-18d5fa2aff41.jpg)](https://user-images.githubusercontent.com/716461/89875294-acbdad80-dbef-11ea-88bb-18d5fa2aff41.jpg)

#### 3.3 分析结果

修改Summation mode 为Total times，即可看到这个方法被哪些上游调用到，调用量和占比

[![4](https://user-images.githubusercontent.com/716461/89875250-9dd6fb00-dbef-11ea-8111-dc92977e61a4.jpg)](https://user-images.githubusercontent.com/716461/89875250-9dd6fb00-dbef-11ea-8111-dc92977e61a4.jpg)

## 总结

1. 通过Arthas profiler命令生成`jfr`文件；
2. 在本地通过`jprofiler`来分析`jfr`文件，定位谁在调用我；
3. 运用之妙，存乎一心。工具的互相结合，可以产生奇妙的化学反应。