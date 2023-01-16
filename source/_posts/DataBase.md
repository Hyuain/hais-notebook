---
title: 数据库
date: 2021-03-20 11:08:35
tags:
  - 入门
categories:
  - [全栈]
---

什么是数据库、Node.js 连接数据库、SQL、范式、表设计、缓存字段、事务、存储引擎

<!-- more -->

# Startup

## 什么是数据库

### 数据库 Database

> 将大量数据保存起来，通过计算机加工而成的可以进行高效访问的数据集合称为数据库

根据数据保存的格式不同，数据库一般被分为：

- 关系型数据库
- 面向对象数据库、XML 数据库、键值存储系统（Redis）、层次数据库

### 数据库管理系统 DBMS

> 用来管理数据库的系统被称为数据库管理系统

- 比如 MySQL、PostgreSQL、SQL Server、DB2、Oracle

DBMS 大概结构如下：

```text
数据库（硬盘/内存中存数据的地方）
           ↑ 读/写
Server 服务端（数据提供者）
 请求的数据 ↓ ↑ SQL语句 
Client 客户端（数据使用者）
```

我们使用的 MySQL 命令，就是一个客户端，MySQL 背后其实还有一个 Server 在 24 小时不间断运行着

## 创建数据库

[下载 MySQL](https://dev.mysql.com/downloads/mysql/)、[下载 MySQL WorkBench](https://dev.mysql.com/downloads/workbench)

使用 Docker {% post_link Docker %}

## 用 Node.js 连接数据库

可以使用 [mysqljs 库](https://github.com/mysqljs/mysql)

可以使用 Sequelize.js

```javascript
const mysql = require("mysql")
const connection = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "123456",
})

connection.connect()

connection.query("CREATE DATABASE IF NOT EXISTS harvey DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;", function (error, results, fields) {
  if (error) throw error
  console.log("创建数据库", results)
})

connection.query("USE harvey;")

connection.query(`CREATE TABLE IF NOT EXISTS user(
  name text,
  age int
);`, function (error, results, fields) {
  if (error) throw error
  console.log("创建表", results)
})

connection.end()
```

# References

## 常见的 SQL 命令

### 操作 database/table

```sql
-- 增
CREATE DATABASE IF NOT EXISTS harvey DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
-- （MySQL 才需要使用 utf8mb4，并且在后面写 COLLATE）

-- 删
DROP database_name;

-- 查
use lzblog;
```

### 操作记录

```sql
-- 增
INSERT INTO users (name, age, `password`) VALUES ('harvey', 20, 123);

-- 删
DELETE FROM users WHERE name='harvey'
-- 一般采用软删除
UPDATE users SET state='0' WHERE username='harvey'
SELECT * FROM users WHERE state<>'0'

-- 改
UPDATE users SET age=24 WHERE name='harvey'
-- You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column.
SET SQL_SAFE_UPDATES = 0

-- 查
SELECT * FROM users LIMIT 10;
SELECT id, name FROM users;
SELECT * FROM users WHERE name='harvey' AND `password`='123';
SELECT * FROM users WHERE name='harvey' OR `password`='123';
SELECT * FROM users WHERE name LIKCE '%ve%' ORDER BY id DESC;
```



- 增：`INSERT INTO user (name, age) VALUES ('harvey', 20);`
- 删：`DELETE FROM user WHERE name='harvey';`
- 改：`UPDATE user SET age=24 WHERE name='harvey';`
- 查：`SELECT * FROM user LIMIT 10;`

#### 用 JOIN 将表连接起来

有时候我们需要拆表、建立关联表等，这时候怎样将数据连起来查询呢？使用 JOIN。

![JOIN](https://www.runoob.com/wp-content/uploads/2019/01/sql-join.png)

##### INNER JOIN (JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，但是只显示 columnA、columnB 两列中有交集的部分。

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-inner.html)

##### LEFT JOIN (LEFT OUTER JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证左边的表（table1，或者说 columnA）数据显示完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-left.html)

##### RIGHT JOIN (RIGHT OUTER JOIN)

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证右边的表（table2，或者说 columnB）数据显示完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-right.html)

##### FULL JOIN (FULL OUTER JOIN)

> MySQL 中不支持 FULL OUTER JOIN

借助两个表中含义相同的列（table 1 中的 columnA、table2 中的 columnB），将两个表关联起来，保证两边的数据都完整，不存在的记录设为 NULL。

```sql
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.columnA=table2.columnB;
```

例子可以看[这里](https://www.runoob.com/sql/sql-join-full.html)

## MySQL 的数据类型

五大类：数字、字符串、时间和日期、JSON（5.7.8 以上）、其他特殊类型

### 数字类型

- bit、tinyint、bool/boolean、smallint、mediumint、int、bigint、decimal、float、double

- serial：一个自增长的非负大整数（BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE）

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)

### 字符串类型

- char(100)、varchar(100)、binary(1024)、varbinary(1024)、blob、text、enum('v1','v2')、set('v1','v2')

varchar 长度可变，可以节省空间

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html)

### 时间和日期

- date、time、datetime、timestamp、year

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-type-syntax.html)

一般传给前端需要转换为 ISO 8601 格式

# Concepts

## 关系型数据库的范式

### 第一范式 1NF

> **字段不可再分**。

比如我们需要存储体检者的双眼视力，那么左右眼视力应该分别存在两个字段中。

### 第二范式 2NF

> 在第一范式的基础上，要有 **键**（键可由多个字段组合）。
> 所有字段分别 **完全依赖** 于键。
> 如果键是多个字段的组合，则 **不允许部分依赖** 于该键。

> 依赖关系：给出键，就能确定唯一字段的值。

比如给出“学号”，就能唯一确定“姓名”，反之则不行，则称“姓名”依赖于“学号”。

比如有一个表中有“学号”“姓名”“课名”“分数”，分数依赖于“学号+课名”，因此将“学号+课名”作为键，但字段“姓名”不依赖于“课名”却依赖于“学号”（“姓名”部分依赖于“学号+课名”）这就不满足第二范式。

### 第三范式 3NF

> 一个表里面不能有两层依赖。

比如一个表中有“学号”“姓名”“系名”“系主任”，“姓名”依赖于“学号”，而“系主任”依赖于“系名”，这就不满足第三范式。

### BC范式

> 键中的属性也不存在间接依赖。

## 数据库设计经验

> 高内聚、低耦合。

### 高内聚

把相关的字段放到一起，不相关的字段分开建表。

如果两个字段能单独建表，那就单独建表。

### 低耦合

如果两个表中有弱关系，采取低耦合

#### 一对一

假设一个学生只能加入一个班级，一个班级里面只能有一个学生。

- 可以把班级放在学生表里面：
  学生表：学生 id、学生姓名、班级 id
  班级表：班级 id、班级名称

- 也可以单独建立关联表：
  学生表：学生 id、学生姓名
  关联表：学生 id、班级 id
  班级表：班级 id、班级名称

#### 一对多

假设一个作者可以写多本数。

若 DBMS 支持数组，可以存两个 id 到一个字段：作者 id、姓名、书（书的 id 数组）

但一般推荐单独建立关系表：

作者表：id、姓名
关联表：作者 id、书 id
书表：id、书名

#### 多对多

假设一个学生可以加入多个班级，一个班级也可以有多个学生。

仍然推荐建立关联表：

学生表：学生 id、学生姓名
关联表：学生 id、班级 id、有效期
班级表：班级 id、班级名称

#### 什么时候建立关联表

当关联自身存在属性时，比如上表的有效期。或者管理多对多这种比较复杂的关系的时候也推荐建立关联表。

# Tricks

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