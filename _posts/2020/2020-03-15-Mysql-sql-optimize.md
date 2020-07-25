---
layout: post
title: "MySQL SQL 优化"
date: "2020-03-15 12:00:00"
category: mysql
tags: mysql
author: Archer
---
* content
{:toc}

## 介绍

MySQL SQL 优化。




## Mysql SQL 优化

### 查看SQL执行频率

```sql
SHOW STATUS LIKE 'Com_%';
```

Com_select：执行SELECT操作的次数，一次查询累加1。其他类似

以下参数只针对InnoDB存储引擎，累加算法略有不同

Innodb_rows_read：SELECT查询操作插入的行数

Innodb_rows_inserted/updated/deleted：执行INSERT/UPDATE/DELETE操作的数

通过以上参数，可以了解当前数据库应用是查询为主还是写入数据为主。

对于事务型的应用。通过Com_commit和Com_rollback可以了解事务提交和回滚的情况对于回滚操作非常的频繁的数据库，可能意味着应用编写存在问题。

基本情况了解：
Connections：试图连接MySQL服务器的次数。
Uptime：服务器工作时间
Slow_queries：慢查询的次数

### 定位执行效率比较低的SQL语句

- 通过慢查询日志定位慢SQL，用--log-slow-queries[=file_name]选项时，mysqld会写一个所有执行时间超过long_query_time秒的SQL语句的日件

- 使用SHOW FULL PROCESSLIST; 查看当前MySQL在进行的线程，同时对一表操作进行优化。

### 通过EXPLAIN分析慢SQL

语法：EXPLAIN SQL语句

- select_type：表示SELECT的类型，常见的取值有SIMPLE(简单表，即不使用表连接或者子查询)、PRIMARY(主查询，即外层的查询)、UNION(UNION中的第二个或者后面的查询语句)、SUBQUERY(子查询中的第一个SELECT)等。

- table：输出结果的表
- type：表示MySQL在表中找到所需行的方式，或者叫访问类
  常见的有：ALL index range ref eq_ref const,system NULL，从右，性能由最差到最好

  type=ALL：全表扫描

  type=index：索引全扫描，MySQL遍历整个索引来查询

  type=range：索引范围扫描，常见于<、<=、>、 >=、 between

  type=ref：使用非唯一索引扫描或唯一索引的前缀扫描，返回匹配某个单独记录

  type=eq_ref：类似ref，区别就在使用的索引是唯一索引，对于每个索引键表中只有一条记录匹配，简单来说，就是多表连接中使用primary key unique index作为关联条件

  type=const/system：单表中最多有一个匹配行，查询起来非常迅速，一般primary key或者唯一索引unique index进行的查询，通过唯一索引uk_ema访问的时候，类型type为const；而从我们构造的仅有一条记录的a表中检索类型type为system
  
  type=NULL：MySQL不用访问表或者索引，就能直接得到结果

  类型type还有其他值，如ref_or_null(与ref类似，区别在于条件中包NULL的查询)、index_merge(索引合并优化)、unique_subquery(in的后一个查询主键字段的子查询)、index_subquery(与unique_subquery类似别在于in的后面是查询非唯一索引字段的子查询
- possible_keys：表示查询时可能使用的索引
- key：表示实际使用的索引
- key_len：使用到索引字段的长度
- rows：扫描行的数
- Extra：执行情况的说明和描述，包含不适合在其他列中显示但是对执行计划重要的额信息
  Using where：表示优化器除了利用索引来加速访问之外，还需要根据索引查询数据

### 通过show profile分析SQL

Show Profile是mysql提供的可以用来分析当前会话中sql语句执行的资源消耗情况的工具，可用于sql调优的测量。默认情况下处于关闭状态，并保存最近15次的运行结果。

#### 查看当前MySQL是否支持profile

```sql
select @@have profiling
```

#### 查看当前MySQL profile参数，默认关闭的

```sql
show variables like 'profiling'
```

#### 开启 profile参数

```sql
set profiling=on
```

### 使用方法

- 执行需要查询SQL
- show profiles; 能看到Query的SQL、Query_ID、Duration
- show profiles for query Query_ID; 能查看到SQL执行过程中每个线程的状态和消耗时间

![SQL执行过程的线程状态和消耗时间](/assets/images/2020/2020-03-15-show-profile-query.png)

> Sending data状态表示MySQL线程开始访问数据并行把结果返回给客户端，而不仅仅是返回结果给客户端。由于在Sending data状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整个查询中耗时最长的状态。

- 详细分析排序

```sql
SELECT
    STATE,
    SUM(DURATION) AS TR,
    ROUND(
* SUM(DURATION) / (
            SELECT
                SUM(DURATION)
            FROM
                information_schema.PROFILING
            WHERE
                QUERY_ID = 3
        ),
    ) AS PR,
    COUNT(*) AS Calls,
    SUM(DURATION) / COUNT(*) AS "R/Call"
FROM
    information_schema.PROFILING
WHERE
    QUERY_ID = 3
GROUP BY
    STATE
ORDER BY
    TR DESC;
```

![SQL执行过程详情](/assets/images/2020/2020-03-15-show-profile-query-detail.png)

- 进一步获取all、cpu、block io、context switch、page faults等明细类型来查看MySQL在使用什么资源上耗费了过高的时间，例如，选择查看CPU的耗费时间。

此时可获取到sending data时间主要消耗在CPU上

![SQL执行过程CPU详情](/assets/images/2020/2020-03-15-show-profile-query-cpu.png)

> 提示：InnoDB引擎count(*)没有MyISAM执行速度快，就是因为InnoDB引擎经历了Sending data状态，存在访问数据的过程，而MyISAM引擎的表在executing之后直接就结束查询，完全不需要访问数据。

### 合理添加索引

- 每张表的索引个数建议不超过6个，索引能增快select查询速度，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引
- 联合索引匹配最左原则(leftmost)，索引为(a,b,c)，能匹配查询a,ab,abc的条件查询
- 以%开头的LIKE查询不能利用B-Tree索引
