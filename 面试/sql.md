.1 查询每门课程成绩都大于80分学生的学号
数据库 表 student
name score course
A 85  语文
A 75  数学
A 82  英语
B   75  语文
B   89  数学
B   79  英语
天使美眉90 语文
天使美眉100 数学
天使美眉100 英语

请找出每门课程都超过80分的那个人名字的SQL语句

SQL1:

select name from test.stu
group by name
having count(score) =sum(case  when score>80 then 1 else 0 end )

SQL2:

select name from stu
group by name
having name not in (
select name from stu
where score <80)

SQL3:

select name from test.stu
group by name
having min(score)>=80



================================================

2. 查询课程001的成绩大于课程002成绩的学号

student表：sno（学号），sname（姓名），sex（性别），dept(系） 
course课程表：cno（课程号），课程名（cname） 
sc选课表：sno，cno，grade（成绩）

select cno from sc a inner join (select * from sc where cno=(select cno from course where cname='001')) as b on a.cno>o=(select cno from course where cname='002')






================================================

3、关于group by表内容：

2005-05-09 胜
2005-05-09 胜
2005-05-09 负
2005-05-09 负
2005-05-10 胜
2005-05-10 负
2005-05-10 负

如果要生成下列结果, 该如何写sql语句

            胜 负
2005-05-09  2  2
2005-05-10  1  2
--------------------------------------------------------
1) select rq,sum(case when shengfu='胜' then 1 else 0 end) as胜,sum(case when shengfu='负' then 1 else 0 end) as负from tab3 group by rq
2) select N.rq,N. 胜,M. 负 from
(select rq,count(*) 胜 from tab3 where shengfu='胜'group by rq)N inner join
(select rq,count(*) 负from tab3 where shengfu='负'group by rq)M on N.rq=M.rq
3) select a.rq,a. 胜  as胜,b.负  as  负from
(select rq,count(shengfu) 胜from tab3 where shengfu='胜' group by rq) a,
(select rq,count(shengfu) 负from tab3 where shengfu='负' group by rq) b
where a.rq=b.rq;
4)select time, sum(decode(status,'胜','')) 胜 ,sum(decode(status,'负','')) 负 from shengfu_table group by time; 　

======================================================


4.表中有A B C三列,用SQL语句实现：当A列大于B列时选择A列否则选择B列，当B列大于C列时选择B列否则选择C列。
select (case when a>b then a else b end),(case when b>c then b else c end) from tab4


5.一个日期判断的sql语句请取出tab5表中日期(SendTime字段)为当天的所有记录 (SendTime字段为datetime型，包含日期与时间)
select * from tab5 t where to_char(t.SendTime,'yyyy-mm-dd')=to_char(sysdate,'yyyy-mm-dd')


6.有一张表，里面有3个字段：语文，数学，英语。其中有3条记录分别表示语文70分，数学80分，英语58分，请用一条sql语句查询出这三条记录并按以下条件显示出来（并写出您的思路）： 
   大于或等于80表示优秀，大于或等于60表示及格，小于60分表示不及格。 
       显示格式： 
       语文              数学                英语 
       及格              优秀                不及格   
-------------------------------------------------------
select
(case when语文>=80 then '优秀' when语文>60 then '及格' else '不及格' end) as 语文,
(case when 数学>=80 then '优秀' when数学>60 then '及格' else '不及格' end) as数学,
(case when英语>=80 then '优秀' when英语>60 then '及格' else '不及格' end) as 英语
from tab5



==================================

7.请用一个sql语句得出结果
从table1,table2中取出如table3所列格式数据

table1

月份mon   部门dep   业绩yj
-------------------------------
一月份      01        10
一月份      02        10
一月份      03         5
二月份      02         8
二月份      04         9
三月份      03         8

table2

部门dep      部门名称depname
--------------------------------
      01      国内业务一部
      02      国内业务二部
      03      国内业务三部
      04      国际业务部

table3 （result）

部门dep     一月份      二月份      三月份
---------------------------------------------------
      01      10        null         null
      02      10         8           null
      03      5         null          8
      04      null       9           null
-------------------------------------------------------
1)
select t.depname,
(select yj from tab6 where mon='一月份' and dep=t.dep) 一月份,
(select yj from tab6 where mon='二月份' and dep=t.dep) 二月份,
(select yj from tab6 where mon='三月份' and dep=t.dep) 三月份
from tab7 t

---------------------
2)求总销售额
select
sum(case when t1.mon='一月份' then t1.yj else 0 end) 一月份,
sum(case when t1.mon='二月份' then t1.yj else 0 end) 二月份,
sum(case when t1.mon='三月份' then t1.yj else 0 end) 三月份
from tab7 t,tab6 t1 where t.dep=t1.dep


8.一个表中的Id有多个记录，把所有这个id的记录查出来，并显示共有多少条记录数。
-------------------------------------------
select id,count(*) from tab8 group by id having count(*)>1

select * from (select tab8,count(id) as num from tab8 group by id) t where t.num>1




9.用一条SQL语句 查询出每门课都大于80分的学生姓名 
8.一个叫department的表，里面只有一个字段name,一共有4条纪录，分别是a,b,c,d,对应四个球对，现在四个球对进行比赛，用一条sql语句显示所有可能的比赛组合.

select t.bh||'vs'||t1.bh from tab10 t,tab10 t1 where t.bh$amp;
select t.bh||'vs'||t1.bh from tab10 t,tab10 t1 where t.bh$amp;t1.bh这个是不分的



=================================
10.怎么把这样一个表儿
year  month amount
1991   1     1.1
1991   2     1.2
1991   3     1.3
1991   4     1.4
1992   1     2.1
1992   2     2.2
1992   3     2.3
1992   4     2.4
查成这样一个结果
year m1  m2  m3  m4
1991 1.1 1.2 1.3 1.4
1992 2.1 2.2 2.3 2.4

a):
select t.year,
(select a.amout from tab11 a where a.month=1 and a.year=t.year) m1,
(select b.amout from tab11 b where b.month=2 and b.year=t.year) m2,
(select c.amout from tab11 c where c.month=3 and c.year=t.year) m3,
(select d.amout from tab11 d where d.month=4 and d.year=t.year) m4
from tab11 t group by t.year


11.拷贝表(拷贝数据,源表名：a 目标表名：b)

SQL: insert into b(a, b, c) select d,e,f from b;

create table test as select * from dept; --从已知表复制数据和结构  

create table test as select * from dept where 1=2; --从已知表复制结构但不包括数据 



12.显示文章、提交人和最后回复时间

  select a.title,a.username,b.adddate from table a,(select max(adddate) adddate from table where table.title=a.title) b


13.日程安排提前五分钟提醒



14.两张关联表，删除主表中已经在副表中没有的信息
  delete from  fubiao a where a.fid not in(select id from zhubiao)


15.有两个表tab12和tab13，均有key和value两个字段，如果tab13的key在tab12中也有，就把tab13的value换为tab12中对应的value

update tab13 set value=(select value from tab12 where tab12.key=tab13.key)



16.原表:
courseid coursename score
-------------------------------------
   1       java      70
   2       oracle    90
   3       xml       40
   4       jsp       30
   5       servlet   80
-------------------------------------
为了便于阅读,查询此表后的结果显式如下(及格分数为60):
courseid coursename score mark
---------------------------------------------------
  1        java     70  pass
  2        oracle   90  pass
  3        xml      40  fail
  4        jsp      30  fail
  5        servlet  80  pass
---------------------------------------------------
select t.courseid,t.coursename,t.score,(case when score>60 then 'pass' else 'fail' end) mark from tab14 t


17.表15
a1   a2
1    a
1    b
2    x
2    y
2    z
用select能选成以下结果吗？
1 ab
2 xyz
SELECT a1, replace(max(sys_connect_by_path(a2, ' ')),' ','') NAME
  FROM (SELECT a1, a2, rn, LAG(rn) OVER(PARTITION BY a1 ORDER BY rn) rn1
          FROM (SELECT a1, a2, row_number() OVER(ORDER BY a1) rn FROM t) rn)
START WITH rn1 IS NULL
CONNECT BY rn1 = PRIOR rn
GROUP BY a1;


18.题为
有两个表, t1, t2,
Table t1:

SELLER | NON_SELLER
-----    -----
  A       B
  A       C
  A       D
  B       A
  B       C
  B       D
  C       A
  C       B
  C       D
  D       A
  D       B
  D       C

Table t2:

SELLER |  BAL
------  --------
  A       100
  B       200
  C       300
  D       400
要求用SELECT 语句列出如下结果:------如A的SUM(BAL)为B,C,D的和,B的SUM(BAL)为A,C,D的和.......
且用的方法不要增加数据库负担,如用临时表等
SELECT SELLER,a.total-t.BAL FROM t,(SELECT SUM(BAL) total FROM t)a;