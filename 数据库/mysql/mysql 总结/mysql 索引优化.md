<style>
</style>

# MySQL索引优化

目录

[MySQL索引优化............................................................................................................... 1](#_Toc15873)

[一、Explain用法........................................................................................................ 1](#_Toc18511)

[二、索引最佳实践..................................................................................................... 8](#_Toc13106)

[1.全值匹配......................................................................................................... 8](#_Toc12048)

[2.最左前缀法则.................................................................................................. 9](#_Toc14354)

[3.不在索引列上做任何操作................................................................................. 9](#_Toc20649)

[4.存储引擎不能使用索引中范围条件右边的列................................................... 10](#_Toc12221)

[5.尽量使用覆盖索引......................................................................................... 10](#_Toc3305)

[6. 尽量不使用不等于（！=或者<>），not in ，not exists.................................... 10](#_Toc26465)

[7.is null,is not null 一般情况下也无法使用索引................................................... 10](#_Toc7796)

[8.like以通配符开头（'%abc...'）mysql索引失效会变成全表扫描操作................. 11](#_Toc4216)

[9.字符串不加单引号索引失效........................................................................... 11](#_Toc29856)

[10. 少用or或in............................................................................................... 11](#_Toc22977)

[11.范围查询优化............................................................................................... 11](#_Toc29954)

[12.联合索引第一个字段用范围不会走索引........................................................ 12](#_Toc1281)

[13.强制走索引.................................................................................................. 12](#_Toc10927)

[14.索引下推...................................................................................................... 13](#_Toc2005)

[15.范围查找特殊情况........................................................................................ 14](#_Toc22591)

[16.Using filesort文件排序原理详解.................................................................... 14](#_Toc30754)

[17.Order by与Group by优化............................................................................. 14](#_Toc19207)

[18.分页查询优化............................................................................................... 20](#_Toc18654)

[19.Join关联查询优化........................................................................................ 21](#_Toc310)

[20.count查询优化............................................................................................. 24](#_Toc16968)

[三、索引设计原则................................................................................................... 24](#_Toc431)

[四、基于慢sql查询做优化...................................................................................... 25](#_Toc14545)

[五、 MySQL数据类型选择...................................................................................... 25](#_Toc31703)

## 一、Explain用法

在 select 语句之前增加 explain 关键字

例如：**explain** select * from actor;

**各个字段含义：**

**1. id****列**

id列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。

id列越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行。

**2. select_type****列**

select_type 表示对应行是简单还是复杂的查询。

1）simple：

简单查询。查询不包含子查询和union

explain select *
from film where id = 2;

2）primary：

复杂查询中最外层的 select

3）subquery：

包含在 select 中的子查询（不在 from 子句中）

4）derived：

包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（

derived的英文含

义）

用这个例子来了解 primary、subquery 和 derived 类型

-- 关闭mysql5.7新特性对衍生表的合并优化

set session
optimizer_switch='derived_merge=off';

explain select
(select 1 from actor where id = 1) from (select * from film where

id = 1) der;

-- 还原默认配置

set session
optimizer_switch='derived_merge=on';

5）union：

在 union 中的第二个和随后的 select

explain select 1
union all select 1;

**3. table****列**

这一列表示 explain 的一行正在访问哪个表。

当 from 子句中有子查询时，table列是 <derivenN> 格式，表示当前查询依赖 id=N 的查询，于是先执行 id=N 的查

询。

当有 union 时，UNION RESULT 的 table 列的值为<union1,2>，1和2表示参与 union 的 select 行id。

**4. type****列 (重要)**

这一列表示**关联类型或访问类型**，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。

依次从最优到最差分别为：**system > const

> eq_ref > ref** **> range > index > ALL**

一般来说，**得保证查询达到range级别，****最好达到ref**

**NULL**：

mysql能够在优化阶段分解查询语句，在执行阶段用不着再访问表或索引。例如：在索引列中选取最小值，可

以单独查找索引来完成，不需要在执行时访问表

explain select min(id) from film;

**const****和system**：

mysql能对查询的某部分进行优化并将其转化成一个常量（可以看show warnings 的结果）。用于

primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。**system是 const的特例**，表里只有一条元组匹配时为system

set session
optimizer_switch='derived_merge=off';

explain extended select * from (select * from film where id = 1) tmp;

show warnings;

**eq_ref**：

primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。这可能是在 const 之外最好的联接类型了。

**ref**：

相比 eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。

1. 简单 select 查询，name是普通索引（非唯一索引）

explain select * from film
where name = 'film1';

2.关联表查询，idx_film_actor_id是film_id和actor_id的联合索引，这里使用到了film_actor的左边前缀film_id部分。

explain select film_id from
film left join film_actor on film.id = film_actor.film_id;

**range**：

范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。使用一个索引来检索给定范围的行。

explain select * from actor
where id > 1;

**index**：

扫描全索引就能拿到结果，一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子节点遍历和扫描，速度还是比较慢的，这种查询一般为使用覆盖索引，二级索引一般比较小，所以这 种通常比ALL快一些。

explain select * from film;

**ALL**：

即全表扫描，扫描你的聚簇索引的所有叶子节点。通常情况下这需要增加索引来进行优化了。

explain select * from
actor;

**5. possible_keys****列**

这一列显示查询可能使用哪些索引来查找。

explain 时可能出现 possible_keys 有列，而 key 显示 NULL 的情况，这种情况是因为表中数据不多，mysql认为索引对此查询帮助不大，选择了全表查询。

如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能，然后用 explain 查看效果。

**6. key****列**

这一列显示mysql实际采用哪个索引来优化对该表的访问。

如果没有使用索引，则该列是 NULL。如果想强制mysql使用或忽视possible_keys列中的索引，在查询中使用 force index、ignore index。

**7. key_len****列**

这一列显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。

举例来说，film_actor的联合索引 idx_film_actor_id 由 film_id 和 actor_id 两个int列组成，并且每个int是4字节。通过结果中的key_len=4可推断出查询使用了第一个列：film_id列来执行索引查找。

explain select * from film_actor where film_id = 2;

key_len计算规则如下：

**字符串**

char(n)和varchar(n)，5.0.3以后版本中，**n均代表字符数，而不是字节数，**如果是utf-8，一个数字或字母占1个字节，一个汉字占3个字节

char(n)：如果存汉字长度就是 3n 字节

varchar(n)：如果存汉字则长度是 3n + 2 字节，加的2字节用来存储字符串长度，因为

varchar是变长字符串

**数值类型**

tinyint：1字节

smallint：2字节

int：4字节

bigint：8字节

**时间类型**

date：3字节

timestamp：4字节

datetime：8字节

如果字段允许为 NULL，需要1字节记录是否为 NULL

索引最大长度是768字节，当字符串过长时，mysql会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索

引。

计算示例：

name` varchar(24)

`age` int(11)

`position` varchar(20)

**Utf-8**

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei';

3 * 24 + 2 = 74

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei' AND age = 22;

（3 * 24 + 2） + 4 = 78

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei' AND age = 22 AND position ='manage

r';

（3 * 24 + 2） + 4 + （3 * 20 + 2） = 140

**非Utf-8，并且name允许为null**

EXPLAIN SELECT * FROM employees_copy
WHERE name= 'LiLei';

1 * 24 + **1** + 2 = 27

EXPLAIN SELECT * FROM employees_copy
WHERE name= 'LiLei' AND age = 22;

（1 * 24 + **1** + 2） + 4 = 31

EXPLAIN SELECT * FROM employees_copy
WHERE name= 'LiLei' AND age = 22 AND position ='manage

r';

（1 * 24 + **1** + 2） + 4 + （1 * 20 + 2） = 53

**8. ref****列**

这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），字段名（例：film.id）

**9. rows****列**

这一列是mysql估计要读取并检测的行数，注意这个不是结果集里的行数。

**10. Extra****列**

这一列展示的是额外信息。常见的重要值如下：

1）**Using
index**：使用覆盖索引

**覆盖索引定义**：mysql执行计划explain结果里的key有使用索引，如果select后面查询的字段都可以从这个索引的树中获取，这种情况一般可以说是用到了覆盖索引，extra里一般都有using index；覆盖索引一般针对的是辅助索引，整个查询结果只通过辅助索引就能拿到结果，不需要通过辅助索引树找到主键，再通过主键去主键索引树里获取其它字段值

explain select film_id from film_actor where film_id = 1;

2）**Using
where**：使用 where 语句来处理结果，并且查询的列未被索引覆盖

explain select * from actor where name = 'a';

3）**Using
index condition**：查询的列不完全被索引覆盖，where条件中是一个前导列的范围；

explain select * from film_actor where film_id > 1;

4）**Using
temporary**：mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。

1. actor.name没有索引，此时创建了张临时表来distinct

explain select distinct name from actor;

2. film.name建立了idx_name索引，此时查询时extra是using index,没有用临时表

explain select distinct name from film;

5）**Using
filesort**：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的。

1. actor.name未创建索引，会浏览actor整个表，保存排序关键字name和对应的id，然后排序name并检索行记录

explain select * from actor
order by name;

2. film.name建立了idx_name索引,此时查询时extra是using
   index

6）**Select tables optimized away**：使用某些聚合函数（比如 max、min）来访问存在索引的某个字段是

explain select min(id) from
film;

## 二、索引最佳实践

;

### 1.全值匹配

SELECT * FROM employees
WHERE name= 'LiLei';

### 2.最左前缀法则

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且**不跳过索引中的列**。

 EXPLAIN SELECT * FROM employees WHERE name = 'Bill' and
age = 31;

 EXPLAIN SELECT * FROM employees WHERE age = 30 AND
position = 'dev';

 EXPLAIN SELECT * FROM employees WHERE position =
'manager';

### 3.不在索引列上做任何操作

**计算、函数、类型转换**，会导致索引失效而转向全表扫描

 EXPLAIN SELECT * FROM employees WHERE left(name,3) =
'LiLei';

给hire_time增加一个普通索引：

ALTER TABLE `employees` ADD
INDEX `idx_hire_time` (`hire_time`) USING BTREE ;

EXPLAIN select * from
employees where date(hire_time) ='2018‐09‐30';

转化为日期范围查询，有可能会走索引：

EXPLAIN SELECT *
FROM employees WHERE hire_time >= '2018-09-30 00:00:00' AND hire_time <=
'2018-09-30 23:59:59';

还原最初索引状态

ALTER TABLE `employees`
DROP INDEX `idx_hire_time`;

### 4.存储引擎不能使用索引中范围条件右边的列

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei' AND age = 22 AND position ='manager';

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei' AND age > 22 AND position ='manager';

### 5.尽量使用覆盖索引

**只访问索引的查询（索引列包含查询列）），减少 select * 语句**

EXPLAIN SELECT name,age FROM
employees WHERE name= 'LiLei' AND age = 23 AND position

='manager';

EXPLAIN SELECT * FROM
employees WHERE name= 'LiLei' AND age = 23 AND position ='manager';

### 6.  尽量不使用不等于（！=或者<>），not in ，not exists

**mysql****在使用不等于（！=****或者<>****），not in** **，not exists** **的时候无法使用索引会导致全表扫描 <** **小于、 >** **大于、 <=****、>=** **这些，mysql****内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引。**

EXPLAIN SELECT * FROM
employees WHERE name != 'LiLei';

### 7.is null,is not null 一般情况下也无法使用索引

EXPLAIN SELECT * FROM
employees WHERE name is null;

### 8.like以通配符开头（'%abc...'）mysql索引失效会变成全表扫描操作

EXPLAIN SELECT * FROM
employees WHERE name like '%Lei';

EXPLAIN SELECT * FROM
employees WHERE name like 'Lei%';

解决like'%字符串%'索引不被使用的方法，使用覆盖索引，查询字段必须是建立覆盖索引字段

EXPLAIN SELECT
name,age,position FROM employees WHERE name like '%Lei%';

### 9.字符串不加单引号索引失效

EXPLAIN SELECT * FROM
employees WHERE name = 1000;

### 10.少用or或in

在使用**or****或in**时，mysql不一定使用索引，mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引。

EXPLAIN SELECT * FROM
employees WHERE name = 'LiLei' or name = 'HanMeimei';

### 11.范围查询优化

给年龄添加单值索引

ALTER TABLE `employees` ADD
INDEX `idx_age` (`age`) USING BTREE;

explain select * from
employees where age >=1 and age <=2000;

没走索引原因：mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引。比如这个例子，可能是由于单次数据量查询过大导致优化器最终选择不走索引

优化方法：可以将大的范围拆分成多个小范围

explain select * from
employees where age >=1 and age <=1000;

explain select * from
employees where age >=1001 and age <=2000;

还原最初索引状态

ALTER TABLE `employees`
DROP INDEX `idx_age`;

### 12.联合索引第一个字段用范围不会走索引

EXPLAIN SELECT * FROM
employees WHERE name > 'LiLei';

联合索引第一个字段就用范围查找不会走索引，mysql内部可能觉得第一个字段就用范围，结果集应该很大，回表效率不高，还不如就全表扫描。

优化策略：**覆盖索引优化**

EXPLAIN SELECT
name,age,position FROM employees WHERE name > 'LiLei' AND age = 22 AND
position ='manager';

### 13.强制走索引

EXPLAIN SELECT * FROM
employees **force** index(idx_name_age_position)
WHERE name > 'LiLei';

‐‐ 关闭查询缓存

set global
query_cache_size=0;

set global
query_cache_type=0;

SELECT * FROM employees
WHERE name > 'LiLei';

set global
query_cache_size=0;

set global
query_cache_type=0;

SELECT * FROM employees **force** index(idx_name_age_position) WHERE name >
'LiLei';

结论：

虽然使用了强制走索引让联合索引第一个字段范围查找也走索引，扫描的行rows看上去也少了点，但是最终查找效率不一定比全表扫描高，因为**回表效率不高。**

### 14.索引下推

EXPLAIN SELECT * FROM
employees WHERE name like 'LiLei%' AND age = 22 AND position ='manager';

MySQL 5.6引入了索引下推优化，**可以在索引遍历过程中，对索引中包含的所有字段先做判断，过滤掉不符合条件的记录之后再回表，可以有效的减少回表次数**。使用了索引下推优化后，上面那个查询在联合索引里匹配到名字是 **'LiLei' 开头**的索引之后，同时还会继续在索引里过滤**age**和**position**这两个字段，拿着过滤完剩下的索引对应的主键id再回表查整行数据。

**为什么范围查找Mysql没有用索引下推优化？**

估计应该是Mysql认为范围查找过滤的结果集过大，like KK% 在绝大多数情况来看，过滤后的结果集比较小，所以这里Mysql选择给 like KK% 用了索引下推优化，当然这也不是绝对的，有时like KK% 也不一定就会走索引下推。

### 15.范围查找特殊情况

EXPLAIN select *
from employees where name > 'aaa';

EXPLAIN select * from employees where name > 'zzz';

Mysql认为name > 'aaa' 结果集很大，不如直接全表扫描，而 name > 'zzz' 结果集就很小了，所以先走索引树，拿到主键再回表查询。

### 16.Using filesort文件排序原理详解

filesort文件排序方式

单路排序：是一次性取出满足条件行的所有字段，然后在sort
buffer中进行排序

双路排序（又叫回表排序模式）：是首先根据相应的条件取出相应的排序字段和可以直接定位行

数据的行 ID，然后在 sort buffer 中进行排序，排序完后需要回表再次取回其它需要的字段；

MySQL 通过比较系统变量 max_length_for_sort_data(默认1024字节) 的大小和需要查询的字段总大小来

判断使用哪种排序模式。

如果字段的总长度小于max_length_for_sort_data ，那么使用 单路排序模式；

如果字段的总长度大于max_length_for_sort_data ，那么使用 双路排序模∙式。

### 17.Order by与Group by优化

Case1：

EXPLAIN select *
from employees where `name` = 'LiLei' and position = 'dev' order by age;

分析：

利用最左前缀法则：中间字段不能断，因此查询用到了name索引，从key_len=74也能看出，age索引列用

在排序过程中，因为Extra字段里没有using filesort

Case 2：

EXPLAIN select *
from employees where `name` = 'LiLei' order by position;

分析：

从explain的执行结果来看：key_len=74，查询使用了name索引，由于用了position进行排序，跳过了

age，出现了Using filesort。

Case 3：

EXPLAIN select *
from employees where `name` = 'LiLei' order by age,position;

分析：

查找只用到索引name，age和position用于排序，无Using filesort。

Case 4：

EXPLAIN select *
from employees where `name` = 'LiLei' order by position,age;

分析：

和Case 3中explain的执行结果一样，但是出现了Using filesort，因为索引的创建顺序为

name,age,position，但是排序的时候age和position颠倒位置了。

Case 5：

EXPLAIN select *
from employees where `name` = 'LiLei' and age = 18 order by position,age;

分析：

与Case 4对比，在Extra中并未出现Using filesort，因为age为常量，在排序中被优化，所以索引未颠倒，

不会出现Using
filesort。

Case 6：

EXPLAIN select *
from employees where `name` = 'LiLei' order by age asc,position desc;

分析：

虽然排序的字段列与索引顺序一样，且order by默认升序，这里position desc变成了降序，导致与索引的

排序方式不同，从而产生Using
filesort。

**age****和position 改成降序**

EXPLAIN select *
from employees where `name` = 'LiLei' order by age desc,position desc;

分析：

这里age和position 变成了降序，mysql会采用从后往前取的方式遍历索引树，拿到主键id后，依次从主键索引取出的结果也是排好序的，所以不会再有Using filesort。

Case 7：

EXPLAIN select *
from employees where `name` in ('LiLei','zhuge') order by age,position;

分析：

对于排序来说，多个相等条件也是范围查询

Case 8：

EXPLAIN select * from employees where `name` > 'a' order by name;

可以用覆盖索引优化

EXPLAIN select
name,age,position from employees where `name` > 'a' order by name;

索引树包含了要查询的所有字段，不用回表了，直接利用索引树排序就可以了。

**Case 9****：**

病案AI审核系统审核管理平台，数据预览-违规费用类型占比接口：

请求url：http://10.128.1.213:8620/app_wheeljack_manager/rest_api/v1/medicare/getDefectTypePer

请求参数：

{

    "hospitalId": "",

    "settlementStartDate": "2020-11-01",

    "settlementEndDate": "2021-11-30"

}

可以看到该接口用时4.84秒，抓取后台sql：

SELECT sum(dt.**reducescroe**) AS illegalAmount,dt.**illegal_type** AS illegalType

FROM 0_detail_tb dt

LEFT JOIN 0_userbase_expand ube ON (

            dt.admissionid = ube.admission_id

            AND ube.hospital_id = dt.hospital_id

)

WHERE dt.**illegal_valid** = 1

AND date_format(ube.settlement_date,'%Y-%m-%d') >= '2020-11-01 00:00:00.0'

AND date_format(ube.settlement_date,'%Y-%m-%d') <= '2021-11-30 00:00:00.0'

GROUP BY dt.**illegal_type** ORDER BY illegalAmount DESC;

Sql执行时间：**2.974**秒

使用EXPLAIN分析：

**key_len: 32 * 3 + 2 + 1 = 99**

0_userbase_expand表数据量：14804

0_detail_tb表数据量：438298

重点优化目标：0_detail_tb表

（1）可以看到查询使用到了illegal_type字段进行分组，所以，先给illegal_type字段建个索引

CREATE INDEX idx_test ON 0_detail_tb (illegal_type);

执行计划没有变化

（2）考虑到最左匹配原则，where条件中还有个illegal_valid字段，继续调整索引

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (illegal_valid,illegal_type);

执行计划没有变化

（3）因为这个表有join查询，所以admissionid和hospital_id也相当于在过滤条件中了，继续调整索引

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (admissionid,hospital_id,illegal_valid,illegal_type);

可以看到**key****列变成了idx_test**，同时**key_len****变成了108** (illegal_typ 列用于排序了，不参与key_len计算)

key_len: (32 * 3 + 2 + 1)  +  4 + (4+1)  = 108

但是，sql执行时间任然需要2秒多

（4）虽然联合索引包含了条件中的所有字段，但是在select 字段中，还有个reducescroe，想到了使用**覆盖索引**，让所查询数据在索引树就可以全部拿到，**减少回表**性能消耗

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (admissionid,hospital_id,illegal_valid,illegal_type,reducescroe);

可以看到，Extra列由null变成了Using index，再次执行sql查询

查询时间变成了0.751秒，时间减少了一大半。

**深入分析：**

**情况1****：**

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (admissionid,hospital_id,illegal_valid,reducescroe,**illegal_type**);

Sql执行时间：0.705秒

**情况2****：**

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (admissionid,hospital_id,**illegal_type,**illegal_valid,reducescroe);

Sql执行时间：0.712秒

**情况3****：**

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (admissionid,**illegal_type,**hospital_id,illegal_valid,reducescroe);

Sql执行时间：0.766秒

**情况4****：**

DROP INDEX idx_test
ON 0_detail_tb;

CREATE INDEX
idx_test ON 0_detail_tb (**illegal_type,**admissionid,hospital_id,illegal_valid,reducescroe);

Sql执行时间：3.467秒

**优化总结：**

1、MySQL支持两种方式的排序filesort和index，Using
index是指MySQL扫描索引本身完成排序。index

效率高，filesort效率低。

2、order
by满足两种情况会使用Using index。

1) order by语句使用索引最左前列。

2) 使用where子句与order
   by子句条件列组合满足索引最左前列。

3、尽量在索引列上完成排序，遵循索引建立（索引创建的顺序）时的最左前缀法则。

4、如果order
by的条件不在索引列上，就会产生Using filesort。

5、能用覆盖索引尽量用覆盖索引

6、group
by与order by很类似，其实质是先排序后分组，遵照索引创建顺序的最左前缀法则。对于group

by的优化如果不需要排序的可以加上**order by null禁止排序**。注意，where高于having，能写在where中

的限定条件就不要去having限定了。

### 18.分页查询优化

**根据自增且连续的主键排序的分页查询**

很多时候我们业务系统实现分页功能可能会用如下sql实现

select * from
employees limit 90000,10;

表示从表 employees 中取出从 90001 行开始的 10 行记录。看似只查询了 10 条记录，实际这条 SQL 是先读取 90010条记录，然后抛弃前 90000 条记录，然后读到后面 10 条想要的数据。因此要查询一张大表比较靠后的数据，执行效率是非常低的。

可以改成如下写法：

EXPLAIN select * from
employees where id > 90000 limit 5;

但是，这条改写的SQL 在很多场景并不实用，这种改写得满足以下两个条件：

（1）主键自增且连续

（2）结果是按照主键排序的

**根据非主键字段排序的分页查询**

再看一个根据非主键字段排序的分页查询，SQL 如下：

select * from
employees ORDER BY name limit 90000,5;

EXPLAIN select *
from employees ORDER BY name limit 90000,5;

发现并没有使用 name 字段的索引（key 字段对应的值为 null），具体原因是：mysql分析出扫描整个索引的成本比扫描全表的成本更高，所以优化器放弃使用索引。

知道不走索引的原因，那么怎么优化呢？

其实关键是让排序时返回的字段尽可能少，所以可以让排序和分页操作先查出主键，然后根据主键查到对应的记录，SQL

改写如下

select * from
employees e inner join (select id from employees order by name limit 90000,5)
ed on e.id=ed.id;

EXPLAIN select * from employees e inner join (select id from
employees order by name limit 90000,5) ed on e.id=ed.id;

需要的结果与原 SQL 一致，执行时间减少了一半以上，我们再对比优化前后sql的执行计划：

原 SQL 使用的是 filesort 排序，而优化后的 SQL 使用的是索引排序。

### 19.Join关联查询优化

**mysql****的表关联常见有两种算法**

**1****、 嵌套循环连接 Nested-Loop Join(NLJ) 算法**

一次一行循环地从第一张表（称为**驱动表**）中读取行，在这行数据中取到关联字段，根据关联字段在另一张表（**被驱动**

**表**）里取出满足条件的行，然后取出两张表的结果合集。

EXPLAIN select *
from t1 inner join t2 on t1.a= t2.a;

从执行计划中可以看到这些信息：

（1）驱动表是 t2，被驱动表是 t1。先执行的就是驱动表(执行计划结果的id如果一样则按从上到下顺序执行sql)；优化器一般会优先选择**小表做驱动表。所以使用 inner join 时，排在前面的表并不一定就是驱动表。**

（2）当使用left join时，左表是驱动表，右表是被驱动表，当使用right join时，右表时驱动表，左表是被驱动表，当使用join时，mysql会选择数据量比较小的表作为驱动表，大表作为被驱动表。

（3）使用了 NLJ算法。一般 join 语句中，如果执行计划 Extra 中未出现 Using join buffer 则表示使用的 join 算法是 NLJ。

**上面sql的大致流程如下：**

1. 从表 t2 中读取一行数据（如果t2表有查询过滤条件的，会从过滤结果里取出一行数据）；

2. 从第 1 步的数据中，取出关联字段 a，到表 t1 中查找；

3. 取出表 t1 中满足条件的行，跟 t2 中获取到的结果合并，作为结果返回给客户端；

4. 重复上面 3 步。

整个过程会读取 t2 表的所有数据(**扫描100行**)，然后遍历这每行数据中字段 a 的值，根据 t2 表中 a 的值索引扫描 t1 表中的对应行(**扫描100次 t1 表的索引，1次扫描可以认为最终只扫描 t1 表一行完整数据，也就是总共 t1 表也扫描了100 行**)。因此整个过程扫描了 **200 行**。

如果被驱动表的关联字段没索引，**使用NLJ算法性能会比较低(下面有详细解释)**，mysql会选择Block Nested-Loop Join算法。

**2****、 基于块的嵌套循环连接 Block Nested-Loop Join(BNL)算法**

把**驱动表**的数据读入到 join_buffer 中，然后扫描**被驱动表**，把**被驱动表**每一行取出来跟 join_buffer 中的数据做对比。EXPLAIN select * from
t1 inner join t2 on t1.b= t2.b;

Extra 中 的Using
join buffer (Block Nested Loop)说明该关联查询使用的是 BNL 算法。

上面sql的大致流程如下：

1. 把 t2 的所有数据放入到 **join_buffer** 中

2. 把表 t1 中每一行取出来，跟 join_buffer 中的数据做对比

3. 返回满足 join 条件的数据

整个过程对表 t1 和 t2 都做了一次全表扫描，因此扫描的总行数为10000(表 t1 的数据总量) + 100(表 t2 的数据总量) = **10100**。并且 join_buffer 里的数据是无序的，因此对表 t1 中的每一行，都要做 100 次判断，所以内存中的判断次数是

100 * 10000= **100** **万次**。

这个例子里表 t2 才 100 行，要是表 t2 是一个大表，

join_buffer 放不下怎么办呢？

join_buffer 的大小是由参数 join_buffer_size 设定的，默认值是 256k。如果放不下表 t2 的所有数据话，策略很简单，就是**分段放**。

比如 t2 表有1000行记录， join_buffer 一次只能放800行数据，那么执行过程就是先往 join_buffer 里放800行记录，然 后从 t1 表里取数据跟 join_buffer 中数据对比得到部分结果，然后清空 join_buffer ，再放入 t2 表剩余200行记录，再次从 t1 表里取数据跟 join_buffer 中数据对比。所以就**多扫了一次 t1 表**。

**被驱动表的关联字段没索引为什么要选择使用 BNL 算法而不使用 Nested-Loop Join 呢？**

如果上面第二条sql使用 Nested-Loop Join，那么扫描行数为 100 * 10000 = 100万次，这个是**磁盘扫描**。

很显然，用BNL磁盘扫描次数少很多，相比于磁盘扫描，**BNL的内存计算会快得多**。

因此MySQL对于被驱动表的关联字段**没索引**的关联查询，一般都会使用 BNL 算法。如果有索引一般选择 NLJ 算法，有索引的情况下 NLJ 算法比 BNL算法性能更高。

**对于关联sql的优化**

**关联字段加索引**，让mysql做join操作时尽量选择NLJ算法

**小表驱动大表**，写多表连接sql时如果**明确知道**哪张表是小表可以用straight_join写法固定连接驱动方式，省去

mysql优化器自己判断的时间。

s**traight_join****解释**：straight_join功能同join类似，但能让左边的表来驱动右边的表，能改表优化器对于联表查询的执行顺序。

比如：select

* from **t2 straight_join t1** on t2.a = t1.a; 代表指定mysql选着 **t2 表作为驱动表**。

straight_join只适用于inner join，并不适用于left join，right join。（因为left join，right join已经代表指

定了表的执行顺序）

尽可能让优化器去判断，因为大部分情况下mysql优化器是比人要聪明的。使用**straight_join**一定要慎重，因

为部分情况下人为指定的执行顺序并不一定会比优化引擎要靠谱。

### 20.count查询优化

‐‐ 临时关闭mysql查询缓存，为了查看sql多次执行的真实时间

set global
query_cache_size=0;

set global
query_cache_type=0;

EXPLAIN select
count(1) from employees;

EXPLAIN select
count(id) from employees;

EXPLAIN select
count(name) from employees;

EXPLAIN select
count(*) from employees;

4个sql执行计划一样说明这四个sql执行效率应该差不多

**注意：以上4条sql只有根据某个****字段count****不会统计字段为****null****值****的数据行**

**深究执行效率，理论上：**

**count(*)≈count(1)>count(****字段)>count(主键 id)**

（1）count(1)跟count(字段)执行过程类似，不过count(1)不需要取出字段统计，就用常量1做统计，count(字段)还需要取出字段，所以理论上count(1)比count(字段)会快一点。

（2）count(*) 是例外，mysql并不会把全部字段取出来，而是专门做了优化，不取值，按行累加，效率很高，所以不需要用count(列名)或count(常量)来替代 count(*)。

（3）为什么对于count(id)，mysql最终选择二级索引而不是主键聚集索引？因为二级索引相对主键索引存储数据更少，检索性能应该更高，mysql内部做了点优化(应该是在5.7版本才优化)。

## 三、索引设计原则

**1****、代码先行，索引后上**

建完表马上就建立索引吗？

这其实是不对的，一般应该等到主体业务功能开发完毕，把涉及到该表相关sql都要拿出来分析之后再建立

索引。

**2****、联合索引尽量覆盖条件**

比如可以设计一个或者两三个联合索引(尽量少建单值索引)，让每一个联合索引都尽量去包含sql语句里的

where、order
by、group by的字段，还要确保这些联合索引的字段顺序尽量满足sql查询的最左前缀原

则。

**3****、不要在小基数字段上建立索引**

索引基数是指这个字段在表里总共有多少个不同的值，比如一张表总共100万行记录，其中有个性别字段，

其值不是男就是女，那么该字段的基数就是2。

如果对这种小基数字段建立索引的话，还不如全表扫描了，因为你的索引树里就包含男和女两种值，根本没

法进行快速的二分查找，那用索引就没有太大的意义了。

一般建立索引，尽量使用那些基数比较大的字段，就是值比较多的字段，那么才能发挥出B+树快速二分查

找的优势来。

**4****、长字符串我们可以采用前缀索引**

尽量对字段类型较小的列设计索引，比如说什么tinyint之类的，因为字段类型较小的话，占用磁盘空间也会

比较小，此时你在搜索的时候性能也会比较好一点。

当然，这个所谓的字段类型小一点的列，也不是绝对的，很多时候你就是要针对varchar(255)这种字段建立

索引，哪怕多占用一些磁盘空间也是有必要的。

对于这种varchar(255)的大字段可能会比较占用磁盘空间，可以稍微优化下，比如针对这个字段的前20个

字符建立索引，就是说，对这个字段里的每个值的前20个字符放在索引树里，类似于 KEY

index(name(20),age,position)。

此时你在where条件里搜索的时候，如果是根据name字段来搜索，那么此时就会先到索引树里根据name

字段的前20个字符去搜索，定位到之后前20个字符的前缀匹配的部分数据之后，再回到聚簇索引提取出来

完整的name字段值进行比对。

但是假如你要是order
by name，那么此时你的name因为在索引树里仅仅包含了前20个字符，所以这个排

序是没法用上索引的， group by也是同理。所以这里大家要对前缀索引有一个了解。

**5****、where与order by冲突时优先where**

在where和order by出现索引设计冲突时，到底是针对where去设计索引，还是针对order by设计索引？到

底是让where去用上索引，还是让order by用上索引?一般这种时候往往都是让where条件去使用索引来快速筛选出来一部分指定的数据，接着再进行排序。

因为大多数情况基于索引进行where筛选往往可以最快速度筛选出你要的少部分数据，然后做排序的成本可

能会小很多。

## 四、基于慢sql查询做优化

可以根据监控后台的一些慢sql，针对这些慢sql查询做特定的索引优化。

关于慢sql查询不清楚的可以参考这篇文章：https://blog.csdn.net/qq_40884473/article/details/89455740

## 五、MySQL数据类型选择

**1****、数值类型**

| 类型          | 大小                              | 范围（有符号）                                                                                                                                   | 范围（无符号）                                                               | 用途       |
| ----------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | -------- |
| TINYINT     | 1 字节                            | (-128, 127)                                                                                                                               | (0, 255)                                                              | 小整数值     |
| SMALLINT    | 2 字节                            | (-32 768, 32 767)                                                                                                                         | (0, 65 535)                                                           | 大整数值     |
| MEDIUMINT   | 3 字节                            | (-8 388 608, 8 388 607)                                                                                                                   | (0, 16 777 215)                                                       | 大整数值     |
| INT或INTEGER | 4 字节                            | (-2 147 483 648, 2 147 483 647)                                                                                                           | (0, 4 294 967 295)                                                    | 大整数值     |
| BIGINT      | 8 字节                            | (-9 233 372 036 854 775 808, 9 223 372 036 854<br> 775 807)                                                                               | (0, 18 446 744 073 709 551 615)                                       | 极大整数值    |
| FLOAT       | 4 字节                            | (-3.402 823 466 E+38, 1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)                                                   | 0, (1.175 494 351 E-38, 3.402 823 466<br> E+38)                       | 单精度 浮点数值 |
| DOUBLE      | 8 字节                            | (1.797 693 134 862 315 7 E+308, 2.225 073 858<br> 507 201 4 E-308), 0, (2.225 073 858 507 201 4 E-308, 1.797 693 134<br> 862 315 7 E+308) | 0, (2.225 073 858 507 201 4 E-308, 1.797<br> 693 134 862 315 7 E+308) | 双精度 浮点数值 |
| DECIMAL     | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                                                                                                                  | 依赖于M和D的值                                                              | 小数值      |

**优化建议**

1.如果整形数据没有负数，如ID号，建议指定为UNSIGNED无符号类型，容量可以扩大一倍。

2.建议使用TINYINT代替ENUM、BITENUM、SET。

3.避免使用整数的显示宽度(参看文档最后)，也就是说，不要用INT(10)类似的方法指定字段显示宽度，直接用INT。

4.DECIMAL最适合保存准确度要求高，而且用于计算的数据，比如价格。但是在使用DECIMAL类型的时候，注意长度设置。

5.建议使用整形类型来运算和存储实数，方法是，实数乘以相应的倍数后再操作。

6.整数通常是最佳的数据类型，因为它速度快，并且能使用AUTO_INCREMENT。

**2****、日期和时间**

| 类型        | 大小 (字节) | 范围                                        | 格式                  | 用途           |
| --------- | ------- | ----------------------------------------- | ------------------- | ------------ |
| DATE      | 3       | 1000-01-01 到 9999-12-31                   | YYYY-MM-DD          | 日期值          |
| TIME      | 3       | '-838:59:59' 到 '838:59:59'                | HH:MM:SS            | 时间值或持续时间     |
| YEAR      | 1       | 1901 到 2155                               | YYYY                | 年份值          |
| DATETIME  | 8       | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值     |
| TIMESTAMP | 4       | 1970-01-01 00:00:00 到 2038-01-19 03:14:07 | YYYYMMDDhhmmss      | 混合日期和时间值，时间戳 |

**优化建议**

1.MySQL能存储的最小时间粒度为秒。

2.建议用DATE数据类型来保存日期。MySQL中默认的日期格式是yyyy-mm-dd。

3.用MySQL的内建类型DATE、TIME、DATETIME来存储时间，而不是使用字符串。

4.当数据格式为TIMESTAMP和DATETIME时，可以用CURRENT_TIMESTAMP作为默认（MySQL5.6以后），MySQL会自动返回记录插入的确切时间。

5.TIMESTAMP是UTC时间戳，与时区相关。

6.DATETIME的存储格式是一个YYYYMMDD HH:MM:SS的整数，与时区无关，你存了什么，读出来就是什么。

7.除非有特殊需求，一般的公司建议使用TIMESTAMP，它比DATETIME更节约空间，但是像阿里这样的公司一般会用DATETIME，因为不用考虑TIMESTAMP将来的时间上限问题。

8.有时人们把Unix的时间戳保存为整数值，但是这通常没有任何好处，这种格式处理起来不太方便，我们并不推荐它。

**3****、字符串**

| 类型         | 大小                | 用途                                                            |
| ---------- | ----------------- | ------------------------------------------------------------- |
| CHAR       | 0-255字节           | 定长字符串，char(n)当插入的字符数不足n时(n代表字符数)，插入空格进行补充保存。在进行检索时，尾部的空格会被去掉。 |
| VARCHAR    | 0-65535 字节        | 变长字符串，varchar(n)中的n代表最大字符数，插入的字符数不足n时不会补充空格                   |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串                                            |
| TINYTEXT   | 0-255字节           | 短文本字符串                                                        |
| BLOB       | 0-65 535字节        | 二进制形式的长文本数据                                                   |
| TEXT       | 0-65 535字节        | 长文本数据                                                         |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据                                                |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                                                      |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据                                                  |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                                                        |

**优化建议**

1.字符串的长度相差较大用VARCHAR；字符串短，且所有值都接近一个长度用CHAR。

2.CHAR和VARCHAR适用于包括人名、邮政编码、电话号码和不超过255个字符长度的任意字母数字组合。那些要用来计算的数字不要用VARCHAR类型保存，因为可能会导致一些与计算相关的问题。换句话说，可能影响到计算的准确性和完整性。

3.尽量少用BLOB和TEXT，如果实在要用可以考虑将BLOB和TEXT字段单独存一张表，用id关联。

4.BLOB系列存储二进制字符串，与字符集无关。TEXT系列存储非二进制字符串，与字符集相关。

5.BLOB和TEXT都不能有默认值。

**PS****：INT显示宽度**

我们经常会使用命令来创建数据表，而且同时会指定一个长度，如下。但是，这里的长度并非是TINYINT类型存储的最大长度，而是显示的最大长度。

如下建表sql语句：

CREATE TABLE
`user`( `id` TINYINT(2) UNSIGNED );

这里TINYINT(2)中2的作用就是，当需要在查询结果前填充0时，命令中加上ZEROFILL就可以实现，如：

`id` TINYINT(2)
UNSIGNED ZEROFILL

这样，查询结果如果是5，那输出就是05。如果指定TINYINT(5)，那输出就是00005，其实实际存储的值还是5，而且存储的数据不会超过255，只是MySQL输出数据时在前面填充了0。

换句话说，在MySQL命令中，字段的类型长度TINYINT(2)、INT(11)不会影响数据的插入，**只会在使用ZEROFILL时有用，让查询结果前填充0**。

navicat不显示前边的0，需要cmd命令行查看
