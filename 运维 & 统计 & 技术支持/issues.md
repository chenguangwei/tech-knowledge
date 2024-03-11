issues
========

记一次服务器从16G内存降级到8G内存后OOM问题

解决办法是 修改 vm.Overcommit =1 

为什么要这样改 参考资料

- [linux下overcommit_memory的问题](https://blog.csdn.net/houjixin/article/details/46412557)

注意理解 overcommit三个值的不同含义

    它是 内存分配策略
    
    可选值：0、1、2。
    0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
    1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
    2， 表示内核允许分配超过所有物理内存和交换空间总和的内存

修改方法：

    有三种方式修改内核参数，但要有root权限：
    
       （1）编辑/etc/sysctl.conf ，改vm.overcommit_memory=1，然后sysctl -p 使配置文件生效
    
      （2）sysctl vm.overcommit_memory=1
    
      （3）echo 1 > /proc/sys/vm/overcommit_memory

 如果出现这种情况 确认是不是需要改这个overcommit的值，可以通过下面方式确认

     grep -i commit /proc/meminfo
    
     看到CommitLimit和Committed_As参数。
    
     CommitLimit是一个内存分配上限,CommitLimit = 物理内存 * overcommit_ratio(默认50，即50%) + swap大小
    
     Committed_As是已经分配的内存大小。
    
     如果 Committed_As-CommitLimit 没有值，两者数值差不多，没有剩余的值了，如果使用 free -m 还有物理内存的话，那就可以把值设为1 看下 