---
title: 数据库
date: 2021-03-20 11:08:35
categories:
  - [全栈]
mathjax: true
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

属性类型可以分为：

- **SImple & Composite**：比如 ID 是简单属性，Address 是复合属性（包含门牌号、街道、城市、国家等）
- **Single-Valued & Multi-Valued**：比如 Age 是 Single-Valued，Phone Numbers 是 Multi-Valued（人可以有很多电话号码）
- **Derived**：比如 Age 可以从 DateOfBirth 推算出来。

属性一般要求是 **原子的（atomic）**，即不可分的。比如整数、字符串是原子的，集合和数组是非原子的。

NULL 是一个特殊值，所有域都有，表示某个值是 unkown。

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

# Relational Algebra

基本操作符：

| 含义               | 操作符       |
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

## Select

$$
\sigma_p(r)
$$

其中 $p$ 称为选择谓词（Selection Predicate），比如：
$$
\sigma_{dept\_name="Physics"}(instrucotr)
$$
可以在其中加入比较运算符 $= \neq > < \geq \leq$ ，逻辑运算符 与 $\land$ 或 $\lor$ 非 $\neg$：
$$
\sigma_{dept\_name="Physics" \land salary>9000}(instrucotr)
$$

## Project

>  只会留下指定的列

$$
\Pi_{A_1, A_2, \dots, A_k}(r)
$$

$A_1, A_2, \dots, A_k$ 是需要的属性名，比如
$$
\Pi_{ID, name, salary}(instructor)
$$

## Composition

关系代数操作符的运算结果是 Relation（表），因此可以将几个操作符像函数一样组合起来：
$$
r''=G(F(r)):r'=F(r);r''=G(r')
$$
比如：
$$
\Pi_{name}(\sigma_{dept\_name="Physics"}(instrucotr))
$$

## Cartesian Product

> 将两个表的数据组合起来

$$
r_1 \times r_2
$$

比如：
$$
instructor \times teaches
$$
![Cartesian Product](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/RelationalAlgebraCartesianProduct.png)

## Join

> 两个表按照规则有意义的组合起来（Catersian Product 会产生很多无意义的数据），相当于只取了匹配规则的值。

$$
r \Join_\theta s = \sigma_\theta(r \times s)
$$

比如：
$$
instructor \Join_{instructor.id=teaches.id} teaches =
\sigma_{constructor.id=teaches.id}(instructor \times teaches)
$$

## Union

$$
r \cup s
$$

$r, s$ 必须有同样多的属性，并且属性的列必须一一对应，比如第二列与第二列类型相同，比如：

重复的行会被删掉。
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \land year=2017}(section)) \cup \Pi_{source\_id}(\sigma_{semester="Spring" \land year=2018}(section))
$$

## Intersection

要求与 Union 相同，比如：
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \land year=2017}(section)) \cap \Pi_{source\_id}(\sigma_{semester="Spring" \land year=2018}(section))
$$

## Set Difference

要求与 Union 相同。比如：
$$
\Pi_{source\_id}(\sigma_{semester="Fall" \land year=2017}(section)) - \Pi_{source\_id}(\sigma_{semester="Spring" \land year=2018}(section))
$$

## Assignment

$$
variable \larr E
$$

## Rename

$$
\rho_X(E)
$$

其中 $E$ 是旧名字，$X$ 是新名字，比如：
$$
\rho_{MyEmployee}(Employee)
$$

# Structured Query Language (SQL)

SQL 包括了：

- Data Manipulation Language (DML)：提供操作数据的方法，比如 INSERT、 UPDATE 和 DELETE 等
- Data Definition Language (DDL)：提供描述和管理数据库的方法，比如 CREATE、ALTER 和 DROP 等

## Data Types

### Large-Object Type

> 当查询返回一个大对象时，他不会返回这个对象本身，而是返回一个指针

- **blob**: 大的二进制文件
- **clob**: 一大堆字符的集合

### User-Defined Type

```sql
CREATE TYPE Dollars AS numeric(12,2)

CREATE TABLE department
  (budget Dollars);
```

## Create Table

```sql
CREATE TABLE r
	(A1 D1, A2, D2, ..., An Dn,
	integrity-constraint1,
	...,
  integrity-constriantk)
```

`r` 是表的名字，`Ai` 表示 `r` 的 schema 中属性的名字，`Di` 表示 `Ai` 值域的数据类型，比如：

```sql
CREATE TABLE instructor (
  ID        char(5),
  name      varchar(20),
  dept_name varchar(20),
  salary    numeric(8,2));
```

## Upate Table

### Insert

```sql
INSERT INTO instructor
VALUES ('10211', 'Smith', 'Biology', 66000);

INSERT INTO instructor (ID, name, dept_name)
VALUES ('10211', 'Smith', 'Biology');
```

### Delete

```sql
DELETE FROM student [WHERE Calause];

DELETE FROM instructor
WHERE dept_name IN (SELECT dept_name
                    FROM department
                    WHERE building = 'Watson');

-- 每当删除一个之后，avg 就会变化，所以要提前存好最开始的 avg
DECLARE @avg_salary numeric(8, 2)
SET @avg_salary = (SELECT AVG(salary) FROM instructor)

DELETE FROM instructor
WHERE salary < @avg_salary;
```

### Update

```sql
UPDATE instructor
SET salary = salary * 1.05
WHERE salary < 70000;

UPDATE instructor
SET salary = CASE
               WHEN salary <= 10000 THEN salray * 1.05
               ELSE salary * 1.03
             END;
```

### Drop

```sql
DROP TABLE student;
```

### Alter

> Add/Drop an attribute to a relation

```sql
ALTER TABLE student ADD age numeric(3,0);

ALTER TABLE student DROP age;
```

### Join

Join 实际上就是 Catersian Product，但对输入的数据有些条件，对也会选择性地输出一些属性。

![JOIN](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/sql-join.png)

#### Natrual Join

会匹配所有的共同属性，并且每个共同属性只保留一列。

![Natural Join](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/DS-NaturalJoin.png)

列出学生和他们参与的课程的 course_id：

```sql
SELECT name, course_id
FROM student, takes
WHERE student.ID = takes.ID;
```

$$
\Pi_{name, course_id}(\sigma_{student.id=teaches.id}(student \times takes))
$$

可以使用 `NATRUAL JOIN` 来写：

```sql
SELECT A1, A2, ... An
FROM r1 NATRUAL JOIN r2 NATRUAL JOIN ... NATRUAL JOIN rn
WHERE P (optional);
```

```sql
SELECT name, course_id
FROM student NATRUAL JOIN takes;
```

也可以用 `INNER JOIN` 来写，但是需要用 `ON` 来指定怎样匹配：

```sql
SELECT name, course_id
FROM student, INNER JOIN takes
ON student.ID = takes.ID;
```

$$
\Pi_{name, course_id}(student \Join_{student.id=takes.id} takes)
$$

注意有的属性可能只是名字相同，但并没有实际的联系，这时 `NATURAL JOIN` 就会把他们错误地连接起来，比如：

`student(ID, dept_name), takes(ID, course_id), course(course_id, dept_name)` 中列出所有的 student.name 以及他们参与的 course.title：

```sql
SELECT name, title
FROM student NATRUAL JOIN takes, course
WHERE takes.course_id = course.course_id;
```

下面的写法是错误的：

```sql
SELECT name, title
FROM student NATRUAL JOIN takes NATRUAL JOIN course;
```

因为 student 中有 `dept_name`，course 中也有 `dept_name`，但其含义是不一样的：student 中的指的是 student 的 dept，course 中的指的是 course 的。

如果这样写的话，`NATRUAL JOIN` 会错误地将两个 `dept_name` 配对起来，从而只保留 student.dept_name 和 course.dept_name 相同的行。

可以通过 `JOIN`（大部分数据库系统中 `JOIN` 和 `INNER JOIN` 是等价的） 和 `USING` 来指明需要相同的列，来避免歧义：

```sql
SELECT name, title
FROM (student NATRUAL JOIN takes) JOIN course USING(course_id)
```

#### Inner Join

就是 Relational Algebra 中的 Join 运算。

![Inner Join](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/DS-InnerJoin.png)

`INNER JOIN` 和 `NATRUAL JOIN` 的不同在于：

- `NATRUAL JOIN` 会自动匹配所有的共同属性，并且只保留一个共同属性
- `INNER JOIN` 需要用户指明需要匹配的属性，并且会保留所有的共同属性（包括指定的属性）

此外，注意使用 `NATRUAL/INNER JOIN` 的关系的先后顺序是有影响的，写在左边表的就会出现在结果的左边

#### Outer Join

Natrual Join 和 Inner Join 最后得到的实际上是 **交集**，比如 student 中有 ID 为 70555 的行，但 takes 中没有，如果将他们 Inner Join on ID，则会忽略该行。

Outer Join 则不会忽略这些只有一个表中有，而另一个表中没有的行，具体有三种 Outer Join：LEFT、RIGHT、FULL。

下面用这个例子来讲解三种 Outer Join，以及他们与 Inner Join 的不同：

**Course**

| course_id  | title       | dept_name  | credits |
| ---------- | ----------- | ---------- | ------- |
| BIO-301    | Genetics    | Biology    | 4       |
| CS-190     | Game Design | Comp. Sci. | 4       |
| **CS-315** | Robotics    | Comp. Sci. | 3       |

**Prereq**

| course_id  | prereq_id |
| ---------- | --------- |
| BIO-301    | BIO-101   |
| CS-190     | CS-101    |
| **CS-347** | CS-101    |

注意上述表中，Course 中没有 course_id 为 CS-347 的项，Prereq 中没有 course_id 为 CS-315 的项。

**Course NATRAL JOIN Prereq**

| course_id | title       | dept_name  | credits | prereq_id |
| --------- | ----------- | ---------- | ------- | --------- |
| BIO-301   | Genetics    | Biology    | 4       | BIO-101   |
| CS-190    | Game Design | Comp. Sci. | 4       | CS-101    |

##### Left Outer Join

> Relational Algebra 中可以使用符号 ⟕ 表示

Left Outer Join 保证了左边的表是完整的：

| course_id  | title        | dept_name      | credits | prereq_id |
| ---------- | ------------ | -------------- | ------- | --------- |
| BIO-301    | Genetics     | Biology        | 4       | BIO-101   |
| CS-190     | Game Design  | Comp. Sci.     | 4       | CS-101    |
| **CS-315** | **Robotics** | **Comp. Sci.** | **3**   | **NULL**  |

##### Right Outer Join

> Relational Algebra 中可以使用符号 ⟖ 表示

Right Outer Join 保证了右边的表是完整的：

| course_id  | title       | dept_name  | credits  | prereq_id  |
| ---------- | ----------- | ---------- | -------- | ---------- |
| BIO-301    | Genetics    | Biology    | 4        | BIO-101    |
| CS-190     | Game Design | Comp. Sci. | 4        | CS-101     |
| **CS-347** | **NULL**    | **NULL**   | **NULL** | **CS-101** |

##### Full Outer Join

> Relational Algebra 中可以使用符号 ⟗ 表示

Right Outer Join 保证了两个表的完整性，*R* ⟗ *S* = (*R* ⟕ *S*) ∪ (*R* ⟖ *S*)：

| course_id  | title        | dept_name      | credits  | prereq_id  |
| ---------- | ------------ | -------------- | -------- | ---------- |
| BIO-301    | Genetics     | Biology        | 4        | BIO-101    |
| CS-190     | Game Design  | Comp. Sci.     | 4        | CS-101     |
| **CS-315** | **Robotics** | **Comp. Sci.** | **3**    | **NULL**   |
| **CS-347** | **NULL**     | **NULL**       | **NULL** | **CS-101** |

## Query

```sql
SELECT A1, A2, ..., An
FROM r1, r2, ..., rm
WHERE P
```

`Ai` 是属性，`ri` 是表，`P` 是查询谓词。注意 SQL 是大小写不敏感的。

### SELECT Clause

可以使用 `DISTINCT` 去掉重复的行，比如：

```sql
SELECT DISTINCT dept_name
FROM instructor;
```

`*` 表示所有属性。

注意也可以使用字符串字面量：

```sql
-- 返回一行包含 '123' 的数据表，表名为 '123'
SELECT '123';
-- 可以给这个返回的数据表重命名
SELECT '123' AS FOO;
-- 返回与 instructor 行数一样多的数据表
SELECT '123' FROM instructor;
```

可以使用算术运算符，也可以使用 `AS` 来重命名：

```sql
SELECT ID, name, salary/12 AS monthly_salary
FROM instructor;
```

### WHERE Clause

可以在 `WHERE` 语句中使用逻辑运算符和比较运算符，`<>` 表示不等于：

```sql
SELECT name
FROM instructor
WHERE dept_name = 'Comp.Sci.' AND salary > 70000;
```

此外还可以使用 `BETWEEN`（两端都包含）：

```sql
SELECT name
FROM instructor
WHERE salary BETWEEN 9000 AND 10000;
```

元组比较：

```sql
SELECT name, course_id
FROM instructor, teaches
WHERE (instructor.ID, dept_name) = (teaches.ID, 'Biology')
-- 这里 instructor.ID 和 teaches.ID 比较，dept_name 和 'Biology' 比较
```

### FROM Clause

`FROM` 会使用对应表的 Cartesian Product，比如下面是在 $instructor \times teaches$ 中进行查找：

```sql
SELECT *
FROM instructor, teaches;
```

通过 Cartesian Product，可以得到所有可能的两表的组合，单独使用可能没什么用，但是经常会与 `WHERE` 语句一同使用：

```sql
SELECT *
FROM Products, Categories
WHERE Products.CategoryID = Categories.CategoryID;
```

### Rename Operation

使用 `AS` 可以对关系（表）或者属性重命名：

找到 Comp. Sci. 中所有的至少比一个 instructor 工资高的 instructor.name。

```sql
SELECT DISTINCT T.name
FROM instructor AS T, instructor AS S
WHERE T.salary > S.salray AND S.dept_name = 'Comp. Sci.'
```

### String Operation

可以用 `LIKE` 来进行字符串匹配，并且有两个通配符：

- `%` 表示任意字串
- `_` 表示任意字符

比如，在所有的 `instructor` 中找到名字包含 `dar` 的：

```sql
SELECT name
FROM instructor
WHERE name LIKE '%dar%';
```

如果我们真的想搜索百分号，我们可以使用 `ESCAPE`：

```sql
LIKE '100\%' ESCAPE '\'
```

通过指定 `\` 为 Escape Character，告诉系统 `\%` 是 `%` 字面量，而不是通配符。

还有一些特殊的用法：

```sql
-- 匹配有且只有三个字符的项
'___'

-- 匹配至少有三个字符的项
'___%'
```

### Ordering

默认会按字母升序（`ASC`）排列，可以指定 `DESC` 让其按照降序排列：

```sql
SELECT DISTINCT name
FROM instructor
ORDER BY name
```

也可以根据多个字段进行排序：

```sql
ORDER BY dept_name, name
ORDER BY salary ASC, name DESC
```

### Set Operation

```sql
(SELECT course_id FROM section WHERE sem = 'Fall' AND year = 2017)
UNION
(SELECT course_id FROM section WHERE sem = 'Spring' AND year = 2018)
```

还有交集 `INTERSECT`，差 `EXCEPT`。

Set Operation 会自动去除重复的项，如果需要保留重复项，可以使用：

```sql
UNION ALL
INTERSECT ALL
EXCEPT ALL
```

### NULL

所有与 `NULL` 的计算都会返回 `NULL`，比如 `5 + NULL` 会返回 `NULL`。

`IS NULL` 可以用来检查是否是 `NULL`：

```sql
SELECT name
FROM instructor
WHERE salary IS NULL;
```

### Aggregate Function

>  聚合函数可以执行计算并返回单个值作为结果： `AVG` `MIN` `MAX` `SUM` `COUNT`

得到 COMP instructors 的平均工资：

```sql
SELECT AVG(salary)
FROM instructor
WHERE dept_name = 'Comp.Sci.';
```

得到在 Spring 2018 教书的 instructors 数量：

```sql
SELECT COUNT(DISTINCT ID)
FROM teaches
WHERE semester = 'Spring' AND year = 2018;
```

得到行的总数：

```sql
SELECT COUNT(*)
FROM course;
```

使用 `GROUP BY` 可以进行分组，比如找到每个系的 instructors 的平均工资：

```sql
SELECT dept_name, AVG(salary)
FROM instructor
GROUP BY dept_name;
```

注意在 Aggregate Functions 之外的 `SELECT` 语句中的属性必须要出现在 `GROUP BY` 中，比如：

```sql
-- error
SELECT dept_name, ID, AVG(salary)
FROM instructor
GROUP BY dept_name;

-- correct
SELECT dept_name, ID, AVG(salary)
FROM instructor
GROUP BY dept_name, ID;
```

#### HAVING Clause

> 用于筛选分组

同为筛选，不过与 `WHERE` 有所不同：

- `WHERE` 是用来筛选行的，`HAVING` 是用来筛选分组的
- `WHERE` 放在 `FROM` 之后，`HAVING` 放在 `GROUP BY` 之后

比如，下面这条语句可以查询 `sales_table` 表中每个产品类别的总销售额：

```sql
SELECT category, SUM(sales) AS total_sales
FROM sales_table
GROUP BY category;
```

如果只选择总销售额大于 1000 的产品类别，可以使用 `HAVING` 来过滤结果：

```sql
SELECT category, SUM(sales) AS total_sales
FROM sales_table
GROUP BY category
HAVING SUM(sales) > 1000;
```

如果进选择销售额发生在 2019 的产品类别，可以在 `WHERE` 中添加筛选条件：

```sql
SELECT category, SUM(sales) AS total_sales
FROM sales_table
WHERE YEAR(sales_date) = 2019
GROUP BY category
HAVING SUM(sales) > 1000;
```

### Nested Subquery

```sql
SELECT A1, A2, ..., An
FROM r1, r2, ..., rm
WHERE P
```

其中 `r1` 可以是子查询（一个 SELECT-FROM-WHERE 表达式），`Ai` 可以是产生一个值的子查询，`P` 则可以按照以下规则定义：

```text
B <operation> (subquery)
```

#### Subqueries in WHERE

找到在 Fall 2017 和 Spring 2018 开的课（使用 `IN`）：

```sql
SELECT DISTINCT course_id
FROM section
WHERE semester = 'Fall' AND year = 2017 AND
  course_id IN (SELECT course_id
                FROM section
                WHERE semester = 'Spring' AND year = 2018);
```

找到在 Fall 2017 但不在 Spring 2018 开的课（使用 `NOT IN`）：

```sql
SELECT DISTINCT course_id
FROM section
WHERE semester = 'Fall' AND year = 2017 AND
  course_id NOT IN (SELECT course_id
                    FROM section
                    WHERE semester = 'Spring' AND year = 2018);
```

找到比至少一个 Biology department 的 instructor 工资更高的 instructors 的名字：

```sql
SELECT DISTINCT name
FROM instructor
WHERE salary > (
  SELECT MIN(salary)
  FROM instructor
  WHERE dept_name = 'Biology'
);

-- 可以这样写
SELECT DISTINCT T.name
FROM instructor as T, instructor as S
WHERE T.salary > S.salary AND S.dept name = 'Biology';

-- 效果等同于用 SOME
SELECT DISTINCT name
FROM instructor
WHERE salray > SOME (SELECT salary
                     FROM instructor
                     WHERE dept_name = 'Biology');
```

`SOME` 表示至少属于对于其中一个成立，比如：

```sql
(5 < SOME {0, 5, 6}) = ture   -- 5 < 6
(5 < SOME {0, 5}) = false     -- 5 > 0 且 5 = 5
(5 = SOME {0, 5}) = true      -- 5 = 5
(5 != SOME {0, 5}) = true     -- 5 != 0
```

找到比所有 Biology department 的 instructor 工资都高的 instructors 的名字：

```sql
SELECT DISTINCT name
FROM instructor
WHERE salary > (SELECT MAX(salary)
                FROM instructor
                WHERE dept_name = 'Biology');
                
-- 效果等同于用 ALL
SELECT DISTINCT name
FROM instructor
WHERE salray > ALL (SELECT salary
                    FROM instructor
                    WHERE dept_name = 'Biology');
```

`ALL` 表示至少对于所有的都成立。

#### Subqueries in FROM

从 departments 的平均工资中找到哪些高于 42000 的：

```sql
SELECT dept_name, avg_salary
FROM (SELECT dept_name, AVG(salary) AS avg_salary
      FROM instructor
      GROUP BY dept_name)
WHERE avg_salary > 42000;

-- 也可以这样写
SELECT dept_name, avg_salary
FROM (SELECT dept_name, AVG(salary)
      FROM instructor
      GROUP BY dept_name
      AS dept_avg(dept_name, avg_salary))
WHERE avg_salary > 42000;
```

#### WITH

可以用 `WITH` 定义临时关系，比如查找所有跟最大 budget 相同的 department：

```sql
WITH max_budget(value) AS
  (SElECT MAX(budget) FROM department)
SELECT department.name
FROM department, max_budget
WHERE department.budget = max_budget.value;
```

找到所有总 salary 高于平均的 department：

```sql
WITH dept_total(dept_name, value) AS
       (SELECT dept_name, SUM(salary)
        FROM instructor
        GROUP BY dept_name),
     dept_total_avg(value) AS
       (SELECT AVG(value)
        FROM dept_total)
SELECT dept_name
FROM dept_total, dept_total_avg
WHERE dept_total.value > dept_total_avg.value;
```

## View

View 是给某人看的虚拟的表，通常可以用来对特定的用户隐藏一些特定的信息。

### View Definition

> 定义 View 并不是创建了一个新的表，而只是保存了这个表达式。

```sql
CREATE VIEW V as <query expression>
```

```sql
CREATE VIEW faculty AS
  SELECT ID, name, dept_name
  FROM instructor;

SELECT name
FROM faculty
WHERE dept_name = 'Biology'

CREATE VIEW departments_total_salary(dept_name, total_salary) AS
  SELECT dept_name, SUM(salary)
  FROM instructor
  GROUP BY dept_name;
```

View 可以用来定义另一个 View，于是形成了 View Chain，如果 View 依赖了自己，则称为递归。

View Exapnsion：比如 View v1 可以写作 Expression e1，e1 中可能也使用了一个 View，将里面的 View 展开成 Expression 的过程称为   View Expansion。

### Materialized View

> 一些数据库系统允许将 View 的结果存下来，他们的结果存下来之后就不会改变了，这种 View 称为 Materialized View

```sql
CREATE MATERIALIZED VIEW faculty AS
  SELECT ID, name, dept_name
  FROM instructor;
```

### View Update

他会将数据插入到原来的表中，比如：

```sql
CREATE VIEW faculty AS
  SELECT ID, name, dept_name
  FROM instructors;
```

```sql
INSERT INTO faculty
VALUES ('30765', 'Green', 'Music');
```

如果 instructors 中有更多的属性，比如 salary，那么可以：

1. 拒绝这次操作；
2. 将新行的 salary 设置为 NULL（更广泛使用）。

```sql
CREATE VIEW history_instructors AS
  SELECT *
  FROM instructors
  WHERE dept_name = 'History';
```

如果我们往 hisoty_instructors 中插入 `('25566', 'Brown', 'Biology', 10000)`，那么 instructors 中会出现这行，而 history_instructors 中则不会。

## Integrity Constraints

完整性约束（比如 `PRIMARY KEY` `NOT NULL` 等）保证了数据的一致性，确保数据库里面的值是合法的。

可以在创建数据库的时候设置：

```sql
CREATE TABLE instructor (
  ID          char(5),
  name        varchar(20) NOT NULL,
  dept_name   varchar(20),
  salary      numeric(8,2),
  PRIMARY KEY (ID),
  FOREIGN KEY (dept_name) REFERENCES department(dept_name));
```

也可以通过这样增加：

```sql
ALTER TABLE table_name ADD constraint;
```

### Constraints on a Single Relation

- 主键约束：`PRIMARY KEY (A1, A2,..., An)` 指定主键。

- 非空约束：`NOT NULL` 表示某个属性不能为 `NULL`。

- 唯一约束：`UNIQUE (A1, A2, ... Am)` 指定这些属性形成了一个 Candidate Key，注意 Candidate Keys 是可以为 NULL 的（Primary Key 不能）。

- 检查约束：`CHECK (P)`

  ```sql
  CREATE TABLE section
    (course_id varchar(8),
     semester varchar(8),
     CHECK (semester in ('Fall', 'Winter', 'Spring', 'Summer')))
  ```
  
  - `CHECK (P)` 中的 P 可以是任意的选择谓词，比如 `CHECK (time_slot_id IN (SELECT time_slot_id FROM time_slot))`
    - 这个条件不只在行插入或者修改的时候检查，也会在 `time_slot` 表改变的时候进行检查

### Referential Integrity

外键约束：外键是指一个表中的一个字段，它引用另一个表中的主键。外键约束确保数据的引用完整性，防止引用不存在的记录。

A 是一组属性，R 和 S 是两个表，他们都含有 A，并且 A 是 S 的主键。如果 R 中出现的所有 A 都在 S 中出现，那么 A 称为 R 的外键。

```sql
FOREIGN KEY (A1, A2,..., An) REFERENCES r
```

比如：

```sql
FOREIGN KEY (dept_name) REFERENCES department
```

默认情况下，外键引用了另外一个表的主键。SQL 也允许具体指定哪些属性：

```sql
FOREIGN KEY (dept_name) REFERENCES (dept_name)
```

正常情况下，如果外键约束被违反了，此次操作会被拒绝。

如果使用 `CASCADE`，那么当主表中的行被删除之后，所有引用了该指的表都会被删除，比如：

```sql
CREATE TABLE course(
  ...
  dept_name varchar(20),
  FOREIGN KEY (dept_name) REFERENCES department
  ON DELETE CASCADE
  ON UPDATE CASCADE,
  ...
)
```

除了 `CASCADE` 以外，也可以使用 `SET NULL` 或者 `SET DEFAULT`。

## Index

一些查询只会涉及到表中的一小部分数据，此时对整个表都进行查找的话效率是很低的，可以通过属性上的 `INDEX` 来让查找的时候只查找这部分数据：

```sql
CREATE INDEX <name> ON <relation-name> (attribute);
```

```sql
CREATE TABLE student
  (ID varchar(5),)
```

# Entity Relationship Model (ERM)

ERM 主要用来帮助设计设计库，描述表之间的关系，独立于数据库硬件和软件的具体实现，主要有三个核心概念：

- **Entity Sets**: 实体集
  - Entity 是客观存在并可相互区别的事物。每个 Entity 可以表示为一组属性，比如 `instrucot = (ID, name, street, city, salary)`，`course  = (course_id, title, credits)`。
  - Entity Set 是一堆同样类型、同样属性的实体的集合，比如 Persons、Cities、Movies。
  - 一些属性组成了实体集的主键，主键可以将实体集中的每个实体区分开来。
- **Relationship Sets**: 关系集
  - Relationship 是一些实体之间的联系。定义 ${(e_1, \cdots ,e_n|e_1 \in E_1, \cdots, e_n \in E_n)}$ 中的 $(e_1, \cdots ,e_n)$ 就是一个关系。
  - Relationship Set 是不同实体集的实体间的数学联系，里面存了好多关系。关系集中除了来自于各个实体的属性之外，还可以拥有自己的属性。
  - Degree of a relationship（关系的度）是关系集中包涉及到的的实体集的数量。
- **Attributes**: 属性

## Cardinality Constraint

**基数约束（Cardinality Constraint）** 是用来表示实体可以有多少实体与另一实体集的实体存在联系，一共有四种形态：

- 1:1（一对一）：Student*WorksOn*Thesis, Department*Has*Dean
- 1:n（一对多）：Building*Has*Room, Lecturer*Teaches*Course
- n:1（多对一）：Room*LocatedIn*Building, Course*ToughtBy*Lecturer
- n:m（多对多）：Student*Takes*Course, Student*Has*Advisor

区分 1:n/n:1 和 n:m：反过来问题

- A building may have multiple rooms...
  - ... but can room be in multiple buildings? *No -> BuildingHasRoom is 1:n*
- A department can be located in multiple buildings...
  - ... but can a building host multiple departments? *Yes -> DepartmentLocatedInBujilding is n:m*

此外两个来自同一个实体集的实体也可以形成关系：

- Person*MarriedTo*Person (1:1)
- Person*IsFatherOf*Person (1:n)
- Person*Has*Father (n:1)
- Person*IsParentOf*Person (n:m)

## Redundant Attribute

比如有两个实体集：

- Instructor(ID, name, *dept_name*, salary)
- Department(dept_name, buidling, budget)

并且将他们用一个关系集联系起来：Instructor*BelongsTo*Department (ID, dept_name)

**此时 Instructor.dept_name 就是冗余属性，可以去掉。**

## Weak Entity Set

考虑另外一个例子：

- 实体集 Building(building_name, address)
- 实体集 Room(number, *building_name*, capacity)
- 关系集 Room*In*Building(number, building_name)

**这时，如果删除了 Room.building_name，就无法区分 Room 中的每一个房间了。**

**弱实体集（Weak Entity Set）** 是一种特殊的实体集，他没有能有效区分每个实体的属性他需要 **一个额外的关系集** 来帮助他区分每个实体。这个额外的关系集叫做 **识别关系集（Identifying Relation Set）**。

- 弱实体没有主键，相对的，他有一个 **识别实体（Identifying Entity）**和一个 **判别器（Discriminator）**。在上述例子中，Buidling 就是识别实体，number 是判别器。

- 弱实体集的存在 **依赖于（Existence Dependent）** 识别实体集，反过来我们称识别实体集 **拥有（Own）** 这个弱实体集。


## Entity Relationship Diagram (ERD)

实体关系图是 ERM 的图形化表达，以下是一个简单的 ER 图：

> **注意 ERD 可能有很多种画法，要具体问题具体分析。**

**第一步：用矩形表示实体集，矩形里面列出实体集的属性，主键会加上下划线。**

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-1.png)

**第二步：用菱形表示关系集。**

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-2.png)

**第三步：可以加上关系集中的其他属性。**

![img](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-3.png)

### Role

同一个实体集可以出现在同一个关系集中多次，每次可以扮演不同的角色：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-4.png)

图中的 course_id 和 prereq_id 就被称为 Roles。

### Cardinality

用不同类型端点来表示基数约束，其中有箭头的表示一，没有箭头的表示多，比如：

![一个老师对一个学生](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-5.png)

![一个老师对多个学生](C:\Users\Harvey\Desktop\ERDiagram-6.png)

也可以直接写上具体的最小和最大基数：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-8.png)

上图表示每个 student 都有一个 instrucotr (1..1)，每个 instructor 可以指导零或多个 student (0..*)

### Total & Partial Participation

用双线表示 **完全参与**（该实体集中的每个实体都对应了关系集中的至少一条关系），单线表示 **部分参与**（一些实体可能不在关系集中），比如：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-7.png)

每个 student 都必须在 advisor 中出现（每个 student 都必须至少有一个 instructor），而有部分 instructor 可能没有 student。

### Attribute

用缩进表示 Composite Attribute，比如：

```text
name
  first_name
  middle_name
  last_name
address
  street
    street_number
    street_name
    apt_number
  city
  state
  zip
```

用大括号表示 Multi-Valued Attribute，比如：

```text
{ phone_number }
```

用函数表示 Derived Attribute，比如：

```text
age()
```

### Weak Entity Set

用双菱形表示识别关系集 Identifying Relationship Set，用虚下划线表示判别器 Discriminator。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-9.png)

### Specialization

> Specialization 简单来讲就是继承关系，与 Specialization 相对应的概念是 Generalization。

![ERDiagram-10](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-10.png)

- Overlapping & Disjoint

  - Overlapping：Person 可以是 Employee，同时也可以是 Student；

  - Disjoint：Employee 只能是 Instructor 或 Secretary 中的一个。

- Partial & Total Specialization

  - Partial：比如 Employee 可以是 Instructor、Secretary 或者没有其他特异功能的普通 Employee，默认情况；
  - Total：比如 Person 必须是 Employee 或者 Student，不能是别的任何类别，通常需要特殊指定。

### Example

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ERDiagram-11.png)

## Relation Schemas

可以将 ERD 转换为 **关系模式（Relation Schemas）**。

### Rpresenting Entity Set

强实体集直接转换成表：building(<u>name</u>, address)

弱实体集转换成的表需要包含他的识别实体集的主键：room(<u>name</u>, <u>number</u>, capacity)

### Representing Relationship Set

多对多：取实体集的主键，联合起来作为关系集的主键，可以再加一些其他的属性，比如 advisor = (<u>studentID</u>, <u>instructorID</u>, date)

一对多：

- 可以取实体集的主键（并再加一些属性），并且把 *多* 的那个实体集的主键作为关系集的主键，比如 insDept = (<u>insID</u>, deptName)
- 也可以关系集合并到将 *多* 的那边的实体集里面去，比如 instructor = (<u>ID</u>, name, salary, dept_name)

一对一：

- 可以取实体集的主键（并再加一些属性），并且任选一边的主键作为关系集的主键，比如 advisor = (<u>instructorID</u>, studentID) 或者 advisor = (instructorID, <u>studentID</u>)
- 将关系集合并到任意一边去，比如 instructor = (<u>ID</u>, name, salary, *studentID*) 或者 student = (<u>ID</u>, name, totCred, *instructorID*)

### Representing Attribute

- Composite Attributes：展开存放
- Multi-Valued Attributes：忽略，用一个单独的表存放
- Derived Attribute：忽略

#### Multi-Valued Attribute

需要一个单独的表用来表达，比如 instructor 中的 phoneNumber，需要用一个新表：instPhone = (<u>ID</u>, <u>phoneNumber</u>)。

原来的一组值就会变成新表中的单独的几行，比如原来的 instructorID 123 有 phoneNumber 1234 和 5678，那么就需要存为两行 (123, 1234) 和 (123, 5678)。

#### Derived Attribute

我们可以创建 View 来表达，比如 Age：

```sql
CREATE VIEW instructorAge AS
  SELECT ID, NOW() - dateOfBirth AS age
  FROM instructor
```

### Representing Higher Arity Relation

表达多元关系时，可以将多个实体集的主键都放进来，比如 projGuide = (instructorID, studentID, projectID)

### Representing Specialization

表达继承关系时，有两种方法，下列我们用 Person(ID, name, street, city)、Employee(ID, salary)、Student(ID, totCredits) 来举例，Employee 和 Student 都继承了 Person。

**方法一：创建三个表，共享他们的主键，同时公共属性只存在高级的实体中。**

- person = (<u>ID</u>, name, street, city)
- employee = (<u>ID</u>, salary)
- student = (<u>ID</u>, totCredits)

该方法的缺陷是如果要拿到 Employee 和 Student 中的完整信息，需要再去访问 Person 表，相当于一次性要访问一个表。

**方法二：创建三个表，共享他们的主键，公共属性存在于所有实体中。**

- person = (<u>ID</u>, name, street, city)
- employee = (<u>ID</u>, name, street, city, salary)
- student = (<u>ID</u>, name, street, city, totCredits)

缺陷是产生了冗余的信息（如果一个人既是 Employee，也是 Student）

# Quick Start

## 创建数据库

[下载 MySQL](https://dev.mysql.com/downloads/mysql/)、[下载 MySQL WorkBench](https://dev.mysql.com/downloads/workbench)

使用 Docker {% post_link Container %}

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