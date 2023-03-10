---
title: 数据库
date: 2021-03-20 11:08:35
categories:
  - [全栈]
---

什么是数据库、Node.js 连接数据库、SQL、范式、表设计、缓存字段、事务、存储引擎

<!-- more -->

# Introduction

## Database

> 将大量数据保存起来，通过计算机加工而成的可以进行高效访问的数据集合称为数据库

根据数据保存的格式不同，数据库一般被分为：

- 关系型数据库
- 面向对象数据库、XML 数据库、键值存储系统（Redis）、层次数据库

## Database Management System (DBMS)

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

## Data Model

> 定义了数据库的逻辑结构，定义了数据是怎样联系在一起的、他们在系统中是怎样被处理和存储的

### Relational Model

所有的数据都存储在不同的 **表（Table）** 中：

- Attributes / Fileds / Columns
- Tuples / Records / Rows

不同的表可以通过 **公共属性（Common Attribute）** 联系在一起。

### Schema and Instances

- Physical Schema：整体的物理架构。
- Logical Schema：整体的逻辑架构，类似于表的标题。
- Instance：在某一时间点的具体数据。

## Database Engine

> DBMS 用来从数据库创建、读取、更新、删除数据的底层模块

- Storage Manager：与操作系统文件管理器交互，并提供高效的数据存储、读取和更新
  - Authorization and Integrity Manager
  - Transaction Manager
  - File Manager
- Query Processor：包括 DDL 解释器、DML 编译器、查询执行引擎
- Transaction Management：在系统出现错误后，保证数据库的一致性，如果一个事务内的某个操作出现错误，可以将该事务内之前已经执行的操作回滚
  - Concurrency-Control Manager：控制并行事务之间的交互

![DatabaseStructure](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/DatabaseStructure.jpg)



# Relational Model

## Terminologies

### Relation Schema & Instance

> 关系模式定义了表格中的 **属性（Attributes）** 和 **域（Domains）**，关系实例是指关系数据库中具体的数据记录。

定义 $A_1, A_2, \dots, A_n$ 为 Attributes，$R=(A_1,A_2,\dots,A_n)$ 是 Relation Schema，$r(R)$ 表示 R 下的一个 Relation Instance，$t$ 可以用来表示 $r$ 中的一个元组（一行）。

比如：

```text
# A Relation Schema
instructor = (ID, name, dept_name, salary)
# A Relation Instance: A specific instructor table
# An Element: A specific instructor
```

### Attribute & Domain

> 域（Domain）表示某个属性的取值范围，比如 $salary=\{1000,\dots, 10000\}\cup\{null\}$

属性一般要求是 **原子的（atomic）**，即不可分的。比如整数、字符串是原子的，集合和数组是非原子的。

null 是一个特殊值，所有域都有，表示某个值是 unkown。

### Database Schema

> 数据库模式是指一个数据库中存储的数据结构的定义，数据库实例是数据库中数据在某个时间点的快照。数据库模式包括多个关系模式。

### Keys

> 超键（Superkey）是能够唯一标识关系中每一个元素的属性集合，候选键（Candidate Key）是最小的超键。

如果 $K \subseteq R$，且 $K$ 能够确定 $r(R)$ 中的每一个元素，则称 $K$ 为 Superkey，比如 $\{ID, name\}$。

超键去除所有冗余的属性之后就是 Candidate Key，比如 $\{ID\}$，通常我们将 Candidate Key 作为主键（Primary Key）。

外键（Foreign Key）约束指的是确保一个表中的数据必须在另一个表中存在，以维护表之间的引用完整性，比如：

```text
instructor(ID, name, dept_name, salary);
department(depart_name, building, budget);
```

## Relational Query Language

- **Procedural Data Manipulation Language (Procedural DML)**：需要确定需要什么数据，以及 **怎样获取这些数据**，具有更强的编程能力，比如关系代数（Relational Algebra）。
- **Declarative DML (non-Procedural DML)**：不需要指明怎样获取数据，比如 SQL (Structured Query Language)。

### Relational Algebra

基本操作符：

| 含义               | 操作符$$     |
| ------------------ | ------------ |
| Select             | $\sigma$     |
| Project            | $\Pi$        |
| Union              | $\cup$       |
| Intersection       | $\cap$       |
| Difference         | $-$          |
| Cattersian Product | $\times$     |
| Rename             | $\rho$       |
| Join               | $\Join$      |
| Assignment         | $\leftarrow$ |

#### Select

$$
\sigma_p(r)
$$

其中 $p$ 称为选择谓词（Selection Predicate），比如：
$$
\sigma_{dept\_name="Physics"}(instrucotr)
$$
可以在其中加入比较运算符 $= \neq > < \geq \leq$ ，逻辑运算符 与 $\and$ 或 $\or$ 非 $\neg$：
$$
\sigma_{dept\_name="Physics" \and salary>9000}(instrucotr)
$$

#### Project

>  只会留下指定的列

$$
\Pi_{A_1, A_2, \dots, A_k}(r)
$$

$A_1, A_2, \dots, A_k$ 是需要的属性名，比如
$$
\Pi_{ID, name, salary}(instructor)
$$

#### Composition

关系代数操作符的运算结果是 Relation（表），因此可以将几个操作符像函数一样组合起来：
$$
r''=G(F(r)):r'=F(r);r''=G(r')
$$
比如：
$$
\Pi_{name}(\sigma_{dept\_name="Physics"}(instrucotr))
$$

#### Catersian Product

> 将两个表的数据组合起来

$$
r_1 \times r_2
$$

比如：
$$
instructor \times teaches
$$
![Cartesian Product](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/RelationalAlgebraCartesianProduct.png)

#### Join

> 两个表按照规则有意义的组合起来（Catersian Product 会产生很多无意义的数据），相当于只取了匹配规则的值。

$$
r \Join_\theta s = \sigma_\theta(r \times s)
$$

比如：
$$
instructor \Join_{instructor.id=teaches.id} teaches =
\sigma_{constructor.id=teaches.id}(instructor \times teaches)
$$

#### Union

$$
r \cup s
$$

$r, s$ 必须有同样多的属性，并且属性的列必须一一对应，比如第二列与第二列类型相同，比如：
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \and year=2017}(section)) \cup \Pi_{source\_id}(\sigma_{semester="Spring" \and year=2018}(section))
$$

#### Intersection

要求与 Union 相同，比如：
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \and year=2017}(section)) \cap \Pi_{source\_id}(\sigma_{semester="Spring" \and year=2018}(section))
$$

#### Set Difference

要求与 Union 相同。比如：
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \and year=2017}(section)) - \Pi_{source\_id}(\sigma_{semester="Spring" \and year=2018}(section))
$$

#### Assignment

$$
variable \larr E
$$

#### Rename

$$
\rho_X(E)
$$

其中 $E$ 是旧名字，$X$ 是新名字，比如：
$$
\rho_{MyEmployee}(Employee)
$$

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