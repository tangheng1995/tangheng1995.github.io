---
layout: post
title: Mysql hash索引 与 btree索引的区别
subtitle: Mysql hash索引 与 btree索引的区别
date:       2020-03-03 12:00:00
author:     "Archer"
categories: 
tags:
    - mysql
    - index
---

### Mysql hash索引 与 btree索引的区别

索引是帮助mysql获取数据的数据结构。最常见的索引是Btree索引和Hash索引。

不同的引擎对于索引有不同的支持：Innodb和MyISAM默认的索引是Btree索引；而Mermory默认的索引是Hash索引。

我们在mysql中常用两种索引算法BTree和Hash，两种算法检索方式不一样，对查询的作用也不一样。

#### BTree

BTree索引是最常用的mysql数据库索引算法，因为它不仅可以被用在=,>,>=,<,<=和between这些比较操作符上，而且还可以用于like操作符，只要它的查询条件是一个不以通配符开头的常量，例如：

```text
select * from user where name like ‘jack%’;
select * from user where name like ‘jac%k%’;
```

如果一通配符开头，或者没有使用常量，则不会使用索引，例如：

```text
select * from user where name like ‘%jack’;
select * from user where name like simply_name;
```

#### Hash

Hash索引只能用于对等比较，例如=,<=>（相当于=）操作符。由于是一次定位数据，不像BTree索引需要从根节点到枝节点，最后才能访问到页节点这样多次IO访问，所以检索效率远高于BTree索引。

但为什么我们使用BTree比使用Hash多呢？主要Hash本身由于其特殊性，也带来了很多限制和弊端：

Hash索引仅仅能满足“=”,“IN”,“<=>”查询，不能使用范围查询。

联合索引中，Hash索引不能利用部分索引键查询。

对于联合索引中的多个列，Hash是要么全部使用，要么全部不使用，并不支持BTree支持的联合索引的最优前缀，也就是联合索引的前面一个或几个索引键进行查询时，Hash索引无法被利用。

Hash索引无法避免数据的排序操作

由于Hash索引中存放的是经过Hash计算之后的Hash值，而且Hash值的大小关系并不一定和Hash运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算。

Hash索引任何时候都不能避免表扫描

Hash索引是将索引键通过Hash运算之后，将Hash运算结果的Hash值和所对应的行指针信息存放于一个Hash表中，由于不同索引键存在相同Hash值，所以即使满足某个Hash键值的数据的记录条数，也无法从Hash索引中直接完成查询，还是要通过访问表中的实际数据进行比较，并得到相应的结果。

Hash索引遇到大量Hash值相等的情况后性能并不一定会比BTree高

对于选择性比较低的索引键，如果创建Hash索引，那么将会存在大量记录指针信息存于同一个Hash值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据访问，而造成整体性能底下。

1. hash索引查找数据基本上能一次定位数据，当然有大量碰撞的话性能也会下降。而btree索引就得在节点上挨着查找了，很明显在数据精确查找方面hash索引的效率是要高于btree的；
2. 那么不精确查找呢，也很明显，因为hash算法是基于等值计算的，所以对于“like”等范围查找hash索引无效，不支持；
3. 对于btree支持的联合索引的最优前缀，hash也是无法支持的，联合索引中的字段要么全用要么全不用。提起最优前缀居然都泛起迷糊了，看来有时候放空得太厉害；
4. hash不支持索引排序，索引值和计算出来的hash值大小并不一定一致。


Mysql索引主要有两种结构：B+树和hash.

hash:hsah索引在mysql比较少用,他以把数据的索引以hash形式组织起来,因此当查找某一条记录的时候,速度非常快.当时因为是hash结构,每个键只对应一个值,而且是散列的方式分布.所以他并不支持范围查找和排序等功能.

B+树:b+tree是mysql使用最频繁的一个索引数据结构,数据结构以平衡树的形式来组织,因为是树型结构,所以更适合用来处理排序,范围查找等功能.相对hash索引,B+树在查找单条记录的速度虽然比不上hash索引,但是因为更适合排序等操作,所以他更受用户的欢迎.毕竟不可能只对数据库进行单条记录的操作. 


Mysql常见索引有：主键索引、唯一索引、普通索引、全文索引、组合索引

PRIMARY KEY（主键索引）  ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) UNIQUE(唯一索引)     ALTER TABLE `table_name` ADD UNIQUE (`column`)
INDEX(普通索引)     ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) FULLTEXT(全文索引)      ALTER TABLE `table_name` ADD FULLTEXT ( `column` )
组合索引   ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` ) 

Mysql各种索引区别：
普通索引：最基本的索引，没有任何限制
唯一索引：与"普通索引"类似，不同的就是：索引列的值必须唯一，但允许有空值。
主键索引：它 是一种特殊的唯一索引，不允许有空值。 
全文索引：仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时好空间。
组合索引：为了更多的提高mysql效率可建立组合索引，遵循”最左前缀“原则。

Mysql自动使用索引规则：
btree索引
当使用 <，<=，=，>=，>，BETWEEN，IN，!=或者<>，以及某些时候的LIKE才会使用btree索引，因为在以通配符%和_开头作查询时，MySQL不会使用索引。btree索引能用于加速ORDER BY操作
hash索引
当使用=，<=>，IN，IS NULL或者IS NOT NULL操作符时才会使用hash索引，并且不能用于加速ORDER BY操作，并且条件值必须是索引列查找某行该列的整个值

#### 引用
