---
title: MySQL优化-EXPLAIN详解
date: 2020-09-05 12:00:24
tags: MySQL
---
# explain执行计划
explain可以输出SQL语句执行的信息，常用的语法格式为`EXPLAIN SELECT statement`
<!-- more -->
explain输出列如下：
```
id					标识号
select_type			查询类型
table				表名称
partitions			匹配的分区
type				连接类型
possible_keys		可能/可以用到的索引
key					实际选择的索引
key_len				选择的索引的长度
ref					
rows				找到记录需要读取的行数的估计值
extra				额外的信息
```
## id
查询标识，是一个有序的数字。
## select_type
查询类型，值包含以下几种
* SIMPLE 语句没有用union和子查询
* PRIMARY 最外层的查询
* UNION union语句中的第二个查询
* DEPENDENT UNION union语句中的第二个查询，依赖外层查询
* UNION RESULT union语句查询结果
* SUBQUERY select语句中的第一个子查询
* DEPENDENT SUBQUERY select语句中的第一个子查询，依赖外层查询
* DERIVED 派生表
* DEPENDENT DERIVED 派生表，依赖另一个表
* MATERIALIZED 物化子查询
* UNCACHEABLE SUBQUERY 子查询的结果不能被缓存，外部查询每一行需要重新计算
* UNCACHEABLE UNION 不可缓存的union语句中的第二个查询

## type
描述表时如何被连接的，以下按由好到坏列出各个值
* system 表中只有一行
* const 表中最多有一个匹配行，如果将一个主键放置到where后面作为条件查询，mysql优化器就能把这次查询优化转化为一个常量
* eq_ref 表中的某行数据读取需要循环前面表中的行，它用在一个索引的所有部分被连接使用并且索引是UNIQUE或PRIMARY KEY