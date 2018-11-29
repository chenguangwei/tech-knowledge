mysql实战学习摘录
=======

#####一条SQL查询语句是如何执行的
- 问答式学习：
    
        为该讲总结了几个问题, 大家复习的时候可以先尝试回答这些问题检查自己的掌握程度:
        1. 
        MySQL的框架有几个组件, 各是什么作用? 
        2. 
        Server层和存储引擎层各是什么作用?
        3. 
        you have an error in your SQL syntax 这个保存是在词法分析里还是在语法分析里报错?
        4. 
        对于表的操作权限验证在哪里进行?
        5. 
        执行器的执行查询语句的流程是什么样的? 
        

##### 一条SQL更新语句是如何执行的？

    首先客户端通过tcp/ip发送一条sql语句到server层的SQL interface
    2.SQL interface接到该请求后，先对该条语句进行解析，验证权限是否匹配
    3.验证通过以后，分析器会对该语句分析,是否语法有错误等
    4.接下来是优化器器生成相应的执行计划，选择最优的执行计划
    5.之后会是执行器根据执行计划执行这条语句。在这一步会去open table,如果该table上有MDL，则等待。
    如果没有，则加在该表上加短暂的MDL(S)
    (如果opend_table太大,表明open_table_cache太小。需要不停的去打开frm文件)
    6.进入到引擎层，首先会去innodb_buffer_pool里的data dictionary(元数据信息)得到表信息
    7.通过元数据信息,去lock info里查出是否会有相关的锁信息，并把这条update语句需要的
    锁信息写入到lock info里(锁这里还有待补充)
    8.然后涉及到的老数据通过快照的方式存储到innodb_buffer_pool里的undo page里,并且记录undo log修改的redo
    (如果data page里有就直接载入到undo page里，如果没有，则需要去磁盘里取出相应page的数据，载入到undo page里)
    9.在innodb_buffer_pool的data page做update操作。并把操作的物理数据页修改记录到redo log buffer里
    由于update这个事务会涉及到多个页面的修改，所以redo log buffer里会记录多条页面的修改信息。
    因为group commit的原因，这次事务所产生的redo log buffer可能会跟随其它事务一同flush并且sync到磁盘上
    10.同时修改的信息，会按照event的格式,记录到binlog_cache中。(这里注意binlog_cache_size是transaction级别的,不是session级别的参数,
    一旦commit之后，dump线程会从binlog_cache里把event主动发送给slave的I/O线程)
    11.之后把这条sql,需要在二级索引上做的修改，写入到change buffer page，等到下次有其他sql需要读取该二级索引时，再去与二级索引做merge
    (随机I/O变为顺序I/O,但是由于现在的磁盘都是SSD,所以对于寻址来说,随机I/O和顺序I/O差距不大)
    12.此时update语句已经完成，需要commit或者rollback。这里讨论commit的情况，并且双1
    13.commit操作，由于存储引擎层与server层之间采用的是内部XA(保证两个事务的一致性,这里主要保证redo log和binlog的原子性),
    所以提交分为prepare阶段与commit阶段
    14.prepare阶段,将事务的xid写入，将binlog_cache里的进行flush以及sync操作(大事务的话这步非常耗时)
    15.commit阶段，由于之前该事务产生的redo log已经sync到磁盘了。所以这步只是在redo log里标记commit
    16.当binlog和redo log都已经落盘以后，如果触发了刷新脏页的操作，先把该脏页复制到doublewrite buffer里，把doublewrite buffer里的刷新到共享表空间，然后才是通过page cleaner线程把脏页写入到磁盘中
    老师，你看我的步骤中有什么问题嘛？我感觉第6步那里有点问题,因为第5步已经去open table了，第6步还有没有必要去buffer里查找元数据呢?这元数据是表示的系统的元数据嘛,还是所有表的？谢谢老师指正
    
- 2、    
    
        我理解备份就是救命药加后悔药，灾难发生的时候备份能救命，出现错误的时候备份能后悔。事情都有两面性，没有谁比谁好，只有谁比谁合适，完全看业务情况和需求而定。一天一备恢复时间更短，binlog更少，救命时候更快，但是后悔时间更短，而一周一备正好相反。我自己的备份策略是设置一个16小时延迟复制的从库，充当后悔药，恢复时间也较快。再两天一个全备库和binlog，作为救命药,最后时刻用。这样就比较兼顾了
        
- 3、
        昨天上午 恢复别人误操作配置表数据，幸好有xtarbackup凌晨的全量备份，只提取了改表的ibd文件，然后在本地 做了 一个一样的空表，释放该表空间，加载 提取后的ibd文件，提取昨天零晨到九点的binlog文件 筛选改表这个时段的操作记录 增量更新到本地导出csv 导入线上 。binlog太tm重要了
   
   
#####为什么你改了我还看不见？

- 1、总结

        1、务的特性：原子性、一致性、隔离性、持久性
        2、多事务同时执行的时候，可能会出现的问题：脏读、不可重复读、幻读
        3、事务隔离级别：读未提交、读提交、可重复读、串行化
        4、不同事务隔离级别的区别：
        读未提交：一个事务还未提交，它所做的变更就可以被别的事务看到
        读提交：一个事务提交之后，它所做的变更才可以被别的事务看到
        可重复读：一个事务执行过程中看到的数据是一致的。未提交的更改对其他事务是不可见的
        串行化：对应一个记录会加读写锁，出现冲突的时候，后访问的事务必须等前一个事务执行完成才能继续执行
        5、配置方法：启动参数transaction-isolation
        6、事务隔离的实现：每条记录在更新的时候都会同时记录一条回滚操作。同一条记录在系统中可以存在多个版本，这就是数据库的多版本并发控制（MVCC）。
        7、回滚日志什么时候删除？系统会判断当没有事务需要用到这些回滚日志的时候，回滚日志会被删除。
        8、什么时候不需要了？当系统里么有比这个回滚日志更早的read-view的时候。
        9、为什么尽量不要使用长事务。长事务意味着系统里面会存在很老的事务视图，在这个事务提交之前，回滚记录都要保留，这会导致大量占用存储空间。除此之外，长事务还占用锁资源，可能会拖垮库。
        10、事务启动方式：一、显式启动事务语句，begin或者start transaction,提交commit，回滚rollback；二、set autocommit=0，该命令会把这个线程的自动提交关掉。这样只要执行一个select语句，事务就启动，并不会自动提交，直到主动执行commit或rollback或断开连接。
        11、建议使用方法一，如果考虑多一次交互问题，可以使用commit work and chain语法。在autocommit=1的情况下用begin显式启动事务，如果执行commit则提交事务。如果执行commit work and chain则提交事务并自动启动下一个事务。
        12、你可以在 information_schema 库的 innodb_trx 这个表中查询长事务，比如下面这个语句，用于查找持续时间超过 60s 的事务。
        select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
        思考题：
        
        在开发过程中，尽可能的减小事务范围，少用长事务，如果无法避免，保证逻辑日志空间足够用，并且支持动态日志空间增长。监控Innodb_trx表，发现长事务报警。
        