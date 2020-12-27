---
title: MySQL存储引擎InnoDB相关知识
date: 2020-08-16 14:21:21
tags: [MySQL, InnoDB]
---
InnoDB是MySQL一个存储引擎。
<!-- more -->
InnoDB优势：
* 支持事务
* 行级锁
* 没有表都有主键
* 支持外键

查看MySQL默认引擎：
* 使用命令SHOW ENGINES;
* 查询INFORMATION_SCHEMA.ENGINES表，SELECT * FROM INFORMATION_SCHEMA.ENGINES;

创建InnoDB表
```sql
CREATE TABLE t1(a int, b char(20), PRIMARY KEY(a)) ENGINE=InnoDB;
```
如果默认存储引擎是InnoDB，则不需要ENGINE=InnoDB。

## 索引
每个InnoDB表都有一个聚簇索引，通常主键即为聚簇索引。
如果创建表时没有定义主键，InnoDB将定位到的第一个非空的唯一索引作为聚簇索引。
如果表没有主键也没有合适的唯一索引，InnoDB内部会生成一个隐藏的名称为GEN_CLUST_INDEX聚簇索引，该索引包含了row ID值。
聚簇索引保存着行中所有的数据，普通索引值包含聚簇索引的值。

### 索引结构
InnoDB索引使用的数据结构是B树，索引记录被存储到B树的叶子节点中，默认的索引页（节点）大小为16KB。

## 表空间
file-per-table表空间，该表空间包含表数据及索引，默认情况下，如果启用innodb_file_per_table参数，InnoDB创建的表将放到该表空间中，参数未启用，表将放到system表空间。
file-per-table表空间对应.ibd文件，名称为`[表名称].ibd`，如：test.t1，创建的ibd文件路径为mysql/data/test/t1.ibd

undo表空间，该表空间包含undo日志

## 锁
### 共享锁shared(s) lock和排它/独占锁exclusive(x) lock
InnoDB实现了标准的行级锁，其中有两种类型：共享锁shared(s) lock和排它/独占锁exclusive(x) lock
持有共享锁则允许读某行。
持有独占锁则允许更新或删除某行。
如果事务t1在行r上持有共享锁，事务t2对行r的请求处理如下：
+ 如果t2持有s锁，则可以访问。
+ 如果t2持有x锁，则不能访问。

如果t1持有x锁，则t2不能访问行r，必须等待t1释放锁。

## 事务隔离级别
InnoDB提供了标准的四种隔离级别：READ_UNCOMMITED、READ_COMMITED、REPEATABLE_READ、SERIALIZABLE，默认隔离级别是REPEATABLE_READ。
