[TOC]



# TiDB

## **TiDB简介**

TiDB 是 PingCAP 公司自主设计、研发的开源分布式关系型数据库，是一款同时支持在线事务处理与在线分析处理 (Hybrid Transactional and Analytical Processing, HTAP）的融合型分布式数据库产品，具备水平扩容或者缩容、金融级高可用、实时 HTAP、云原生的分布式数据库、兼容 MySQL 5.7 协议和 MySQL 生态等重要特性。目标是为用户提供一站式 OLTP (Online Transactional Processing)、OLAP (Online Analytical Processing)、HTAP 解决方案。TiDB 适合高可用、强一致要求较高、数据规模较大等各种应用场景。

>  引用:https://docs.pingcap.com/zh/tidb/stable/overview

## **什么是HTAP**

混合事务/分析处理（Hybrid Transaction Analytical Processing, HTAP）是数据库技术领域的新名词，是在线事务（OnLine Transaction Processing）和在线分析（Online Analytical Processing）合称简写，即（HTAP = OLAP +OLTP）, HTAP既可以在线交易事务，又可以在线实时分析。

2014年Gartner的一份报告中使用混合事务分析处理(HTAP)一词描述新型的应用程序框架，以打破OLTP和OLAP之间的隔阂，既可以应用于事务型数据库场景，亦可以应用于分析型数据库场景，实现实时业务决策。这种架构具有显而易见的优势：不但避免了繁琐且昂贵的ETL操作，而且可以更快地对最新数据进行分析。这种快速分析数据的能力将成为未来企业的核心竞争力之一。

HTAP的合理性：

- 数据刚进入数据库的时，可称之为“热数据”；热数据在 OLTP 场景下会被频繁修改。此时数据宜行存储。
- 随着数据慢慢变久，数据越来越“冷”；这个时候数据不太可能被频繁的修改，对数据的查询和分析越来越多。此时数据宜列存储。



## **TiDB 是什么？**

TiDB 是一个分布式 NewSQL 数据库。它支持水平弹性扩展、ACID 事务、标准 SQL、MySQL 语法和 MySQL 协议，具有数据强一致的高可用特性，是一个不仅适合 OLTP 场景还适OLAP 场景的混合数据库。

## **TiDB怎么来的？**

开源分布式缓存服务 Codis 的作者，**PingCAP** 联合创始人& CTO ，资深 infrastructure 工程师的**黄东旭**，擅长分布式存储系统的设计与实现，开源狂热分子的技术大神级别人物。即使在互联网如此繁荣的今天，在数据库这片边界模糊且不确定地带，他还在努力寻找确定性的实践方向。

2012 年底，他看到 Google 发布的两篇论文，得到了很大的触动，这两篇论文描述了 Google 内部使用的一个海量关系型数据库 F1/Spanner ，解决了关系型数据库、弹性扩展以及全球分布的问题，并在生产中大规模使用。“如果这个能实现，对数据存储领域来说将是颠覆性的”，黄东旭为完美方案的出现而兴奋， PingCAP 的 TiDB 在此基础上诞生了。

## **TiDB架构**

TiDB在整体架构基本是参考 Google Spanner 和 F1 的设计，上分两层为 **TiDB** 和 **TiKV** 。 TiDB 对应的是 Google F1， 是一层无状态的 SQL Layer ，兼容绝大多数 MySQL 语法，对外暴露 MySQL 网络协议，负责解析用户的 SQL 语句，生成分布式的 Query Plan，翻译成底层 Key Value 操作发送给 TiKV ， TiKV 是真正的存储数据的地方，对应的是 Google Spanner ，是一个分布式 Key Value 数据库，支持弹性水平扩展，自动的灾难恢复和故障转移（高可用），以及 ACID 跨行事务。值得一提的是 TiKV 并不像 HBase 或者 BigTable 那样依赖底层的分布式文件系统，在性能和灵活性上能更好，这个对于在线业务来说是非常重要。

与传统的单机数据库相比，TiDB 具有以下优势：

- 纯分布式架构，拥有良好的扩展性，支持弹性的扩缩容
- 支持 SQL，对外暴露 MySQL 的网络协议，并兼容大多数 MySQL 的语法，在大多数场景下可以直接替换 MySQL
- 默认支持高可用，在少数副本失效的情况下，数据库本身能够自动进行数据修复和故障转移，对业务透明
- 支持 ACID 事务，对于一些有强一致需求的场景友好，例如：银行转账
- 具有丰富的工具链生态，覆盖数据迁移、同步、备份等多种场景

在内核设计上，TiDB 分布式数据库将整体架构拆分成了多个模块，各模块之间互相通信，组成完整的 TiDB 系统。对应的架构图如下：

![architecture](https://download.pingcap.com/images/docs-cn/tidb-architecture-1.png)

- TiDB Server：SQL 层，对外暴露 MySQL 协议的连接 endpoint，负责接受客户端的连接，执行 SQL 解析和优化，最终生成分布式执行计划。TiDB 层本身是无状态的，实践中可以启动多个 TiDB 实例，通过负载均衡组件（如 LVS、HAProxy 或 F5）对外提供统一的接入地址，客户端的连接可以均匀地分摊在多个 TiDB 实例上以达到负载均衡的效果。TiDB Server 本身并不存储数据，只是解析 SQL，将实际的数据读取请求转发给底层的存储节点 TiKV（或 TiFlash）。
- PD Server：整个 TiDB 集群的元信息管理模块，负责存储每个 TiKV 节点实时的数据分布情况和集群的整体拓扑结构，提供 TiDB Dashboard 管控界面，并为分布式事务分配事务 ID。PD 不仅存储元信息，同时还会根据 TiKV 节点实时上报的数据分布状态，下发数据调度命令给具体的 TiKV 节点，可以说是整个集群的“大脑”。此外，PD 本身也是由至少 3 个节点构成，拥有高可用的能力。建议部署奇数个 PD 节点。
- 存储节点
  - TiKV Server：负责存储数据，从外部看 TiKV 是一个分布式的提供事务的 Key-Value 存储引擎。存储数据的基本单位是 Region，每个 Region 负责存储一个 Key Range（从 StartKey 到 EndKey 的左闭右开区间）的数据，每个 TiKV 节点会负责多个 Region。TiKV 的 API 在 KV 键值对层面提供对分布式事务的原生支持，默认提供了 SI (Snapshot Isolation) 的隔离级别，这也是 TiDB 在 SQL 层面支持分布式事务的核心。TiDB 的 SQL 层做完 SQL 解析后，会将 SQL 的执行计划转换为对 TiKV API 的实际调用。所以，数据都存储在 TiKV 中。另外，TiKV 中的数据都会自动维护多副本（默认为三副本），天然支持高可用和自动故障转移。
  - TiFlash：TiFlash 是一类特殊的存储节点。和普通 TiKV 节点不一样的是，在 TiFlash 内部，数据是以列式的形式进行存储，主要的功能是为分析型的场景加速。

>
>
>官网架构参考指南：https://docs.pingcap.com/zh/tidb/dev/tidb-architecture

## **TiDB开发语言**

在 TiDB 研发语言的选择过程中，放弃了 Java 而采用 Go 。TiDB整个项目分为两层，TiDB 作为 SQL 层，采用 Go 语言开发， TiKV 作为下边的分布式存储引擎，采用 Rust 语言开发。在架构上确实类似 FoundationDB，也是基于两层的结构。 FoundationDB 的 SQL Layer 采用 Java ，底层是 C++ ，不过在去年，被 Apple 收购了。 在选择编程语言并没有融入太多的个人喜好偏向， SQL 层选择 Go 相对 Java 来说：

第一是 他们团队的背景使用 Go 的开发效率更高，而且性能尚可，尤其对于高并发程序而言，可以使用 goroutine / channel 等工具用更少的代码写出正确的程序；

第二是 在标准库中很多包对网络程序开发非常友好，这个对于一个分布式系统来说非常重要；

第三是 在存储引擎底层对于性能要求很高，Go 毕竟是一个带有 GC 和 Runtime 的语言，在 TiKV 层可以选择的方案并不多，过去基本只有 C 或 C++，不过近两年随着 Rust 语言的成熟，又在经过长时间的思考和大量实验，最终他们团队选择了 Rust（ Rust是Mozilla开发的注重安全、性能和并发性的编程语言。“Rust”，由web语言的领军人物**Brendan Eich**（js之父），Dave Herman以及Mozilla公司的Graydon Hoare 合力开发。）。

## **与 MySQL 兼容性对比**

TiDB 支持包括跨行事务，JOIN 及子查询在内的绝大多数 MySQL 的语法，用户可以直接使用现有的 MySQL 客户端连接。如果现有的业务已经基于 MySQL 开发，大多数情况不需要修改代码即可直接替换单机的 MySQL。

包括现有的大多数 MySQL 运维工具（如 PHPMyAdmin, Navicat, MySQL Workbench 等），以及备份恢复工具（如 mysqldump, mydumper/myloader）等都可以直接使用。

不过一些特性由于在分布式环境下没法很好的实现，目前暂时不支持或者是表现与 MySQL 有差异。

一些 MySQL 语法在 TiDB 中可以解析通过，但是不会做任何后续的处理，例如 Create Table 语句中 Engine 以及 Partition 选项，都是解析并忽略。更多兼容性差异请参考具体的文档。

**不支持的特性**

存储过程

视图

触发器

自定义函数

外键约束

全文索引

空间索引

非 UTF8 字符集

## **TiDB 基本操作**

下面具体介绍 TiDB 中基本的增删改查操作。

**创建、查看和删除数据库**

```text
使用 CREATE DATABASE 语句创建数据库。语法如下：
CREATE DATABASE db_name [options];
例如，要创建一个名为 samp_db 的数据库，可使用以下语句：
CREATE DATABASE IF NOT EXISTS samp_db;
使用 SHOW DATABASES 语句查看数据库：
SHOW DATABASES;
使用 DROP DATABASE 语句删除数据库，例如：
DROP DATABASE samp_db;
```

**创建、查看和删除表**

```text
使用 CREATE TABLE 语句创建表。语法如下：
CREATE TABLE table_name column_name data_type constraint;
例如：
CREATE TABLE person (
number INT(11),
name VARCHAR(255),
birthday DATE
);
如果表已存在，添加 IF NOT EXISTS 可防止发生错误：
CREATE TABLE IF NOT EXISTS person (
number INT(11),
name VARCHAR(255),
birthday DATE
);
使用 SHOW CREATE 语句查看建表语句。例如：
SHOW CREATE table person;
使用 SHOW FULL COLUMNS 语句查看表的列。 例如：
SHOW FULL COLUMNS FROM person;
使用 DROP TABLE 语句删除表。例如：
DROP TABLE person;
或者
DROP TABLE IF EXISTS person;
使用 SHOW TABLES 语句查看数据库中的所有表。例如：
SHOW TABLES FROM samp_db;
```

**创建、查看和删除索引**

```text
对于值不唯一的列，可使用 CREATE INDEX 或 ALTER TABLE 语句。例如：
CREATE INDEX person_num ON person (number);
或者
ALTER TABLE person ADD INDEX person_num (number);
对于值唯一的列，可以创建唯一索引。例如：
CREATE UNIQUE INDEX person_num ON person (number);
或者
ALTER TABLE person ADD UNIQUE person_num on (number);
使用 SHOW INDEX 语句查看表内所有索引：
SHOW INDEX from person;
使用 ALTER TABLE 或 DROP INDEX 语句来删除索引。与 CREATE INDEX 语句类似，DROP INDEX 也可以嵌入 ALTER TABLE 语句。例如：
DROP INDEX person_num ON person;
ALTER TABLE person DROP INDEX person_num;
```

**增删改查数据**

```text
使用 INSERT 语句向表内插入数据。例如：
INSERT INTO person VALUES("1","tom","20170912");
使用 SELECT 语句检索表内数据。例如：
SELECT * FROM person;
+--------+------+------------+
| number | name | birthday |+--------+------+------------+
| 1 | tom | 2017-09-12 |+--------+------+------------+
使用 UPDATE 语句修改表内数据。例如：
UPDATE person SET birthday='20171010' WHERE name='tom';
SELECT * FROM person;
+--------+------+------------+
| number | name | birthday |+--------+------+------------+
| 1 | tom | 2017-10-10 |+--------+------+------------+
使用 DELETE 语句删除表内数据：
DELETE FROM person WHERE number=1;
SELECT * FROM person;
Empty set (0.00 sec)
```

**创建、授权和删除用户**

```text
使用 CREATE USER 语句创建一个用户 tiuser，密码为 123456：
CREATE USER 'tiuser'@'localhost' IDENTIFIED BY '123456';
授权用户 tiuser 可检索数据库 samp_db 内的表：
GRANT SELECT ON samp_db.* TO 'tiuser'@'localhost';
查询用户 tiuser 的权限：
SHOW GRANTS for tiuser@localhost;
删除用户 tiuser：
DROP USER 'tiuser'@'localhost';
```