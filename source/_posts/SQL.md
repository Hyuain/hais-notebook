---
title: SQL 基本语法
date: 2020-02-04 16:39:35
tags:
  - 入门
  - 慕课网
  - 自学
categories:
  - [全栈, 数据库]
---

涉及 SQL 的基本语法。

<!-- more -->

# 下载

[下载 MySQL](https://dev.mysql.com/downloads/mysql/)
[下载 MySQL WorkBench](https://dev.mysql.com/downloads/workbench)

# 增

```sql
-- use [schemas name]
usz lzblog;

-- insert into [table name] ([colunm name]) values ([colunm value])
-- password 是关键字，所以要用 `` 引起来
insert into users (username, `password`, nickname) values ('zs', '123', 'zhangsan')
```

# 删

```sql
delete from users where username ='zs';

-- 一般采用软删除

update users set state='0' where username='lz';

select * from users where state <> '0';
```

# 改

```sql
update users set nickname='lz2' where username='lz';
-- update [table name] set [colunm name]=[value] where [condition]

-- You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column.
SET SQL_SAFE_UPDATES = 0
```

# 查

```sql
select * from users;
select id,username from users;
select * from users where username='lz' and `password`='123';
select * from users where username='lz' or `password`='123';
select * from users where username like '%z%';
select * from users where username like '%z%' order by id desc;
```