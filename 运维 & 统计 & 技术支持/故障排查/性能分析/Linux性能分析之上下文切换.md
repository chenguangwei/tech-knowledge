# Linux性能分析之上下文切换

而在每个任务运行前，CPU 都需要知道任务从哪里加载、又从哪里开始运行，也就是说，需要系统事先帮它设置好 CPU 寄存器和程序计数器

**CPU 寄存器**，是 CPU 内置的容量小、但速度极快的内存。而程序计数器，则是用来存储CPU 正在执行的指令位置、或者即将执行的下一条指令位置。它们都是 CPU 在运行任何任务前，必须的依赖环境，因此也被叫做 **CPU 上下文**

**CPU 上下文切换**，就是先把前一个任务的 CPU 上下文（也就是 CPU 寄存器和程序计数器）保存起来，然后加
载新任务的上下文到这些寄存器和程序计数器，最后再跳转到程序计数器所指的新位置，运行新任务。

根据任务的不同，CPU 的上下文切换就可以分为几个不同的场景，也就是**进程上下文切换、线程上下文切换以及中断上下文切换**。

#### 进程上下文切换

Linux 按照特权等级，把进程的运行空间分为内核空间和用户空间，CPU 特权等级的 Ring 0 和 Ring 3。进程既可以在用户空间运行，又可以在内核空间中运行。进程在用户空间运行时，被称为进程的用户态，而陷入内核空间的时候，被称为进程的内核态。
从用户态到内核态的转变，需要通过系统调用来完成。比如，当我们查看文件内容时，就需要多次系统调用来完成：首先调用 open() 打开文件，然后调用 read() 读取文件内容，并调用 write() 将内容写到标准输出，最后再调用 close() 关闭文件。
内核空间（Ring 0）具有最高权限，可以直接访问所有资源；
用户空间（Ring 3）只能访问受限资源，不能直接访问内存等硬件设备，必须通过系统调用陷入到内核中，才能访问这些特权资源。

系统调用的过程有没有发生 CPU 上下文的切换呢？答案自然是肯定的

CPU 寄存器里原来用户态的指令位置，需要先保存起来。接着，为了执行内核态代码，
CPU 寄存器需要更新为内核态指令的新位置。最后才是跳转到内核态运行内核任务 >
而系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间，继续运行进程。所以，一次系统调用的过程，其实是发生了**两次 CPU 上下文切换。**

系统调用过程中，并不会涉及到虚拟内存等进程用户态的资源，也不会切换进程。这跟我们通常所说的进程上下文切换是不一样的：

进程上下文切换，是指从一个进程切换到另一个进程运行。
而系统调用过程中一直是同一个进程在运行

所以，系统调用过程通常称为特权模式切换，而不是上下文切换。但实际上，系统调用过程中，CPU 的上下文切换还是无法避免

#### 进程上下文切换跟系统调用又有什么区别呢？

进程是由内核来管理和调度的，**进程的切换只能发生在内核态。**所以，进程的上下文不仅包括了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的状态。
因此，进程的上下文切换就比系统调用时多了一步：在保存当前进程的内核状态和 CPU 寄存器之前，需要先把该进程的虚拟内存、栈等保存下来；而加载了下一进程的内核态后，还需要刷新进程的虚拟内存和用户栈。

如下图所示，保存上下文和恢复上下文的过程并不是“免费”的，需要内核在 CPU 上运行才能完成

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731191404.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731191404.png)

每次上下文切换都需要几十纳秒到数微秒的 CPU 时间。这个时间还是相当可观的，特别是在进程上下文切换次数较多的情况下，很容易导致 CPU 将大量时间耗费在寄存器、内核栈以及虚拟内存等资源的保存和恢复上，进而大大缩短了真正运行进程的时间。这也正是上一节中我们所讲的，导致平均负载升高的一个重要因素。

Linux 通过 TLB（Translation Lookaside Buffer）来管理虚拟内存到物理内存的映射关系。当虚拟内存更新后，TLB 也需要刷新，内存的访问也会随之变慢。特别是在多处理器系统上，缓存是被多个处理器共享的，刷新缓存不仅会影响当前处理器的进程，还会影响共享缓存的其他处理器的进程

#### 什么时候会切换进程上下文

进程切换时才需要切换上下文，换句话说，只有在进程调度的时候，才需要切换上下文。Linux 为每个 CPU 都维护了一个就绪队列，将活跃进程（即正在运行和正在等待CPU 的进程）按照优先级和等待 CPU 的时间排序，然后选择最需要 CPU 的进程，也就是优先级最高和等待 CPU 时间最长的进程来运行

#### 进程在什么时候才会被调度到 CPU 上运行呢？

就是进程执行完终止了，它之前使用的 CPU 会释放出来，这个时候再从就绪队列里，拿一个新的进程过来运行。其实还有很多其他场景，也会触发进程调度，在这里我给你逐个梳理下。

- 其一，为了保证所有进程可以得到公平调度，CPU 时间被划分为一段段的时间片，这些时间片再被轮流分配给各个进程。这样，当某个进程的时间片耗尽了，就会被系统挂起，切
  换到其它正在等待 CPU 的进程运行。

- 其二，进程在系统资源不足（比如内存不足）时，要等到资源满足后才可以运行，这个时候进程也会被挂起，并由系统调度其他进程运行。

- 其三，当进程通过睡眠函数 sleep 这样的方法将自己主动挂起时，自然也会重新调度。

- 其四，当有优先级更高的进程运行时，为了保证高优先级进程的运行，当前进程会被挂起，由高优先级进程来运行。

- 其五，发生硬件中断时，CPU 上的进程会被中断挂起，转而执行内核中的中断服务程序。

#### 线程上下文切换

线程与进程最大的区别在于，线程是调度的基本单位，而进程则是资源拥有的基本单位。
所谓内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚拟内存、全局变量等资源。
可以这么理解线程和进程：

当进程只有一个线程时，可以认为进程就等于线程。

当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源。这些资源在上下文切换时是不需要修改的。

另外，线程也有自己的私有数据，比如栈和寄存器等，这些在上下文切换时也是需要保存的

线程的上下文切换其实就可以分为两种情况：

- 第一种，前后两个线程属于不同进程。此时，因为资源不共享，所以切换过程就跟进程上下文切换是一样。

- 第二种，前后两个线程属于同一个进程。此时，因为虚拟内存是共享的，所以在切换时，虚拟内存这些资源就保持不动，只需要切换线程的私有数据、寄存器等不共享的数据

同进程内的线程切换，要比多进程间的切换消耗更少的资源，而这，也正是多线程代替多进程的一个优势

#### 中断上下文切换

为了快速响应硬件的事件，中断处理会打断进程的正常调度和执行，转而调用中断处理程序，响应设备事件。而在打断其他进程时，就需要将进程当前的状态保存下来，这样在中断结束后，进程仍然可以从原来的状态恢复运行。
跟进程上下文不同，中断上下文切换并不涉及到进程的用户态。所以，即便中断过程打断了一个正处在用户态的进程，也不需要保存和恢复这个进程的虚拟内存、全局变量等用户态资源。

中断上下文，其实只包括内核态中断服务程序执行所必需的状态，包括 CPU 寄存器、内核堆栈、硬件中断参数等。对同一个 CPU 来说中断处理比进程拥有更高的优先级，所以中断上下文切换并不会与进程上下文切换同时发生。同样道理，由于中断会打断正常进程的调度和执行，所以大部分中断处理程序都短小精悍，以便尽可能快的执行结束。另外，跟进程上下文切换一样，中断上下文切换也需要消耗 CPU，切换次数过多也会耗费大量的 CPU，甚至严重降低系统的整体性能。所以，当你发现中断次数过多时，就需要注意去排查它是否会给你的系统带来严重的性能问题

#### 怎么查看系统的上下文切换情况

使用 vmstat 这个工具，来查询系统的上下文切换情况 ,主要关注以下几项

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731211223.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731211223.png)



```
cs（context switch）是每秒上下文切换的次数。
in（interrupt）则是每秒中断的次数。
r（Running or Runnable）是就绪队列的长度，也就是正在运行和等待 CPU 的进程数。
b（Blocked）则是处于不可中断睡眠状态的进程数 
```

另外

free 空闲的物理内存的大小

buff 设备和设备之间的缓冲

Linux/Unix系统是用来存储，目录里面有什么内容，权限等的``缓存

cache cpu和内存之间的缓冲

cache直接用来记忆我们打开的文件,给文件做缓冲，我本机大概占用300多M(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)

si 每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。我的机器内存充裕，一切正常。

so 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。

bi 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
我本机上没什么IO操作，所以一直是0，但是我曾在处理拷贝大量数据(2-3T)的机器上看过可以达到140000/s，磁盘写入速度差不多140M每秒

bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。

us 用户CPU时间
我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。

sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。

id 空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。

wa 等待IO CPU时间。

查看每个进程的详细情况

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731211407.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731211407.png)

cswch ，表示每秒自愿上下文切换(voluntary context switches）的次数，

nvcswch ，表示每秒非自愿上下文切换（non voluntary context switches）的次数

所谓自愿上下文切换，是指进程无法获取所需资源，导致的上下文切换。比如说，
I/O、内存等系统资源不足时，就会发生自愿上下文切换。
而非自愿上下文切换，则是指进程由于时间片已到等原因，被系统强制调度，进而发生
的上下文切换。比如说，大量进程都在争抢 CPU 时，就容易发生非自愿上下文切换

如果出现（cs）性能问题那么主要从最后五个方面排查

大体上分为两方面：

1. cpu 不足
2. 进程或线程自己需要切换

针对cpu不足有两种情况;

- 1.1、本进程消耗 --解决自身问题
- 1.2、其他进程消耗 --解决别人那问题

### 案例分析（1）



1，查看正常情况下的上下文切换情况

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215553.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215553.png)

2，模拟多线程切换

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215734.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215734.png)

3，查看此时的上下文切换情况[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215649.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215649.png)

4，使用pidstat查看上下文切换的情况

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215813.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731215813.png)

分析

1，发现cs列的上下文切换从66已经涨到了87万多

R列：就绪队列长度已经到了7，远远超过了cpu的个数2，所以会产生大量的cpu竞争

us和sy列：cpu使用率加起来已经到100%其中系统 CPU 使用率，也就是 sy 列高达 87%，说明 CPU 主要是被内核占用了

in 列：中断次数也上升到了 3 万左右，说明中断处理也是个潜在的问题

综合这几个指标，可以知道，系统的就绪队列过长，也就是正在运行和等待 CPU 的进程数过多，导致了大量的上下文切换，而上下文切换又导致了系统 CPU 的占用率升高

2，从 pidstat 的输出你可以发现，CPU 使用率的升高果然是 sysbench 导致的，它的 CPU使用率已经达到了 100%。但上下文切换则是来自其他进程，包括非自愿上下文切换频率最高的 pidstat ，以及自愿上下文切换频率最高的内核线程 kworker 和 sshd

3，**使用-wt 参数表示输出线程的上下文切换指标 **

虽然 sysbench 进程（也就是主线程）的上下文切换次数看起来并不多，但它的子线程的上下文切换次数却有很多。看来，上下文切换罪魁祸首，还是过多的sysbench 线程

[![img](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731220856.png)](https://cdn.jsdelivr.net/gh/JasonsteagleWu/picgo_01/20190731220856.png)

4，查看/proc/interrupts ，查看中断使用情况

/proc 实际上是 Linux 的一个虚拟文件系统，用于内核空间与用户空间之间的通信。/proc/interrupts 就是这种通信机制的一部分，提供了一个只读的中断使用情

watch -d cat /proc/interrupts

可以通过vmstat 、 pidstat 和 /proc/interrupts 等工具，来辅助排查性能问题的根源



### 案例分析（2）



#### 一、cpu不足

##### 1.1情况模拟：

使用top命令查看

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009161952406-755137778.png)

看到CPU被消耗光了，确实也都在用户空间us cpu 使用vmstat 查看（CPU空闲时也差不多有2000多的切换）

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009162357576-209228251.png)

从vmstat上来看，并看不到多大的CS。但是已经可以看到CPU队列高、CPU使用率高了

使用命令pidstat -w -t -p 15288 查看

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009164334925-797245988.png)

 

从上图可以看到nvcswch/s已经有值了，我这里是每秒刷新一次。也就是说每个线程大概每秒被动切换了50多次。

你有没有觉得，这个值看起来似乎并不大是不是？并且在vmstat中也没看到多高的CS切换呀，因为在CPU空闲时也差不多有2000多的切换呀。（在我这个环境中是这样的数值，在其他环境中，这个值会有变化。）

 

为什么会出现这种情况呢。因为现在CPU都被15288抢占，本来操作系统正常的CS都抢不到CPU了，只被15288里面的几个进程消耗掉了。那正常的CS就连CPU都抢不到，当然CS也就减少了，这时整个系统其实是处在瘫痪的状态的。

##### 1.2情况模拟：

cpu 被其他进程消耗，导致本进程被动切换

使用top命令查看

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009171118946-1507782889.png)

消耗cpu的进程有两个16550 16551

然后启动我们自己的模拟进程

 

 

 

 

 

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009171330293-407818662.png)

 

查看模拟进程的上下文切换情况：

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009172034512-1663198480.png)

从命令看来cs比平时低很多

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009172657084-1785492310.png)

看到切换了，并且切换的意愿不是很强烈。被动切换

那么现在杀掉其他两个进程 再来查看切换

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009173212755-1186268892.png)

是不是少了很多

 

### 二、进程或线程自己需要切换

自动切换和服务代码有关

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009182825833-222459379.png)

这里启动了400个线程来达到预期结果。 这里可以看到sy cpu 过高 是因为模拟代码中仅仅是sleep 如果失败业务代码的话就是us cpu 了

这里来看一下上下文切换情况：

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009183157149-126954691.png)

这种情况是资源切换上下文的情况。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * context_switch - switch to the new MM and the new thread's register state.
 3  */
 4 static __always_inline struct rq *
 5 context_switch(struct rq *rq, struct task_struct *prev,
 6            struct task_struct *next, struct rq_flags *rf)
 7 {
 8     prepare_task_switch(rq, prev, next);
 9  
10     /*
11      * For paravirt, this is coupled with an exit in switch_to to
12      * combine the page table reload and the switch backend into
13      * one hypercall.
14      */
15     arch_start_context_switch(prev);
16  
17     /*
18      * kernel -> kernel   lazy + transfer active
19      *   user -> kernel   lazy + mmgrab() active
20      *
21      * kernel ->   user   switch + mmdrop() active
22      *   user ->   user   switch
23      */
24     if (!next->mm) {                                // to kernel
25         enter_lazy_tlb(prev->active_mm, next);
26  
27         next->active_mm = prev->active_mm;
28         if (prev->mm)                           // from user
29             mmgrab(prev->active_mm);
30         else
31             prev->active_mm = NULL;
32     } else {                                        // to user
33         membarrier_switch_mm(rq, prev->active_mm, next->mm);
34         /*
35          * sys_membarrier() requires an smp_mb() between setting
36          * rq->curr / membarrier_switch_mm() and returning to userspace.
37          *
38          * The below provides this either through switch_mm(), or in
39          * case 'prev->active_mm == next->mm' through
40          * finish_task_switch()'s mmdrop().
41          */
42         switch_mm_irqs_off(prev->active_mm, next->mm, next);
43  
44         if (!prev->mm) {                        // from kernel
45             /* will mmdrop() in finish_task_switch(). */
46             rq->prev_mm = prev->active_mm;
47             prev->active_mm = NULL;
48         }
49     }
50  
51     rq->clock_update_flags &= ~(RQCF_ACT_SKIP|RQCF_REQ_SKIP);
52  
53     prepare_lock_switch(rq, next, rf);
54  
55     /* Here we just switch the register state and the stack. */
56     switch_to(prev, next, prev);
57     barrier();
58  
59     return finish_task_switch(prev);
60 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里是切换上下文源码。

- context_switch ：切换上下文的方法（下面是切换上下文的三个参数）
- 　　rq ：指向切换上下文发生cpu的运行队列
- 　　prev：被切进程
- 　　next：切向进程

![img](https://img2020.cnblogs.com/blog/1232786/202010/1232786-20201009184824813-1723578926.png)

