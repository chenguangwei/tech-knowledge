mybatis
========

1、mybastis中的 '#' 和'！' 的区别
        
    #{ } 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符。
    例如，sqlMap 中如下的 sql 语句
    select * from user where name = #{name};
    解析为：
    select * from user where name = ?;
    一个 #{ } 被解析为一个参数占位符 ? 。
    
    ${ } 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换
    例如，sqlMap 中如下的 sql
    select * from user where name = '${name}';
    当我们传递的参数为 "ruhua" 时，上述 sql 的解析为：
    
    select * from user where name = "ruhua";
    预编译之前的 SQL 语句已经不包含变量 name 了。
    
    综上所得， ${ } 的变量的替换阶段是在动态 SQL 解析阶段，而 #{ }的变量的替换是在 DBMS 中。
    
    1、能使用 #{ } 的地方就用 #{ }
    2、表名作为变量时，必须使用 ${ }
参考：

- [mybatis深入理解(一)之 # 与 $ 区别以及 sql 预编译](https://segmentfault.com/a/1190000004617028)

- [MyBatis-Plus 指南](https://mp.baomidou.com/guide/faq.html)

- [mybatis源码中文注释](https://github.com/tuguangquan/mybatis)