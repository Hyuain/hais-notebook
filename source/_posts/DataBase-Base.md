---
title: 数据库基本知识 2
date: 2021-03-20 11:08:35
tags:
  - 入门
categories:
  - [全栈, 数据库]
---

范式、表设计、缓存字段、事务、存储引擎

<!-- more -->

# 关系型数据库的范式

## 第一范式 1NF

> **字段不可再分**。

比如我们需要存储体检者的双眼视力，那么左右眼视力应该分别存在两个字段中。

## 第二范式 2NF

> 在第一范式的基础上，要有 **键**（键可由多个字段组合）。
> 所有字段分别 **完全依赖** 于键。
> 如果键是多个字段的组合，则 **不允许部分依赖** 于该键。

> 依赖关系：给出键，就能确定唯一字段的值。
 
比如给出“学号”，就能唯一确定“姓名”，反之则不行，则称“姓名”依赖于“学号”。

比如有一个表中有“学号”“姓名”“课名”“分数”，分数依赖于“学号+课名”，因此将“学号+课名”作为键，但字段“姓名”不依赖于“课名”却依赖于“学号”（“姓名”部分依赖于“学号+课名”）这就不满足第二范式。

## 第三范式 3NF

> 一个表里面不能有两层依赖。

比如一个表中有“学号”“姓名”“系名”“系主任”，“姓名”依赖于“学号”，而“系主任”依赖于“系名”，这就不满足第三范式。

## BC范式

> 键中的属性也不存在间接依赖。

# 数据库设计经验

> 高内聚、低耦合。

## 高内聚

把相关的字段放到一起，不相关的字段分开建表。

如果两个字段能单独建表，那就单独建表。

## 低耦合

如果两个表中有弱关系，采取低耦合

### 一对一

假设一个学生只能加入一个班级，一个班级里面只能有一个学生。

- 可以把班级放在学生表里面：
  学生表：学生 id、学生姓名、班级 id
  班级表：班级 id、班级名称

- 也可以单独建立关联表：
  学生表：学生 id、学生姓名
  关联表：学生 id、班级 id
  班级表：班级 id、班级名称

### 一对多

假设一个作者可以写多本数。

若 DBMS 支持数组，可以存两个 id 到一个字段：作者 id、姓名、书（书的 id 数组）

但一般推荐单独建立关系表：

作者表：id、姓名
关联表：作者 id、书 id
书表：id、书名

### 多对多

假设一个学生可以加入多个班级，一个班级也可以有多个学生。

仍然推荐建立关联表：

学生表：学生 id、学生姓名
关联表：学生 id、班级 id、有效期
班级表：班级 id、班级名称

### 什么时候建立关联表

当关联自身存在属性时，比如上表的有效期。或者管理多对多这种比较复杂的关系的时候也推荐建立关联表。

# 用 JOIN 将表连接起来

有时候我们需要拆表、建立关联表等，这时候怎样将数据连起来查询呢？使用 JOIN。

![JOIN](https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png)

## INNER JOIN (JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，但是只显示 columnA、columnB 两列中有交集的部分。

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-inner.html)

## LEFT JOIN (LEFT OUTER JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证左边的表（table1，或者说 columnA）数据显示完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-left.html)

## RIGHT JOIN (RIGHT OUTER JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证右边的表（table2，或者说 columnB）数据显示完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-right.html)

## FULL JOIN (FULL OUTER JOIN)

> MySQL 中不支持 FULL OUTER JOIN

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证两边的数据都完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-full.html)

# 一些小技巧

## 缓存字段

一个例子：一个 blog 包含多个 comments，如何获取博客的评论数？

```sql
SELECT COUNT(id) FROM comments WHERE blog_id=8
```

这样会比较慢，可以在 blog 表上加一个 comment_count 字段，每次添加 comment 就 +1，删除就 -1

## 事务 Transaction

在上述例子的解决办法中有个问题，用户的评论操作产生了两件事：1. comments 表新增记录；2. blog 表的 comment_count 字段 +1。

这样不能保证两边的数据同步，因为有可能其中一步失败了，这就需要用到事务。如果其中有一条语句失败了，则全部不执行。

```sql
START TRANSACTION;
语句1; 2; 3;
COMMIT;
```

可以看[这个教程](https://www.runoob.com/mysql/mysql-transaction.html)

## MySQL 存储引擎

可以通过 `show engines` 查看 MySQL 支持的存储引擎。

- InnoDB，默认，一般认为是事务性数据库的首选
- MyISAM，拥有较高的插入、查询速度，但不支持事务
- Memory，内存中快速访问数据
- Archive，只支持 insert 和 select

## 索引

```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name(length))
```

可以看[这个教程](https://www.runoob.com/sql/sql-create-index.html)

索引可以提高查询数据的效率，但更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。

因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。