---
title: 数据库基本知识
date: 2021-03-14 10:49:35
tags:
  - 入门
categories:
  - [全栈, 数据库]
---

数据库的基本知识以及用 Docker 来安装数据库。

<!-- more -->

# 用 Docker 来安装数据库

## 安装 Docker

- 进入 [Docker Hub 官方网站](https://hub.docker.com/) 注册并下载
- 确保 `docker --version` 返回版本号
- 在 Docker Engine 中设置国内镜像，[可以点这里查看教程](http://guide.daocloud.io/dcs/daocloud-9153151.html)
```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": []
}
```

## Docker 安装 MySQL

- 进入 Docker 上面 [MySQL 的主页](https://hub.docker.com/_/mysql)
- 运行命令创建容器：
```bash
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
# -d 表示守护进程形式，不会随意关掉
# -p 用来设置端口映射 本机端口号:虚拟机端口号
docker run --name mysql-demo1 -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql:latest
```

常见的 docker 命令：

- `docker run` 启动新容器
- `docker ps -a` 查看所有的容器（Containers）
- `docker kill <id|name>` 关闭对应的容器
- `docker restart <id|name>` 重启关闭的容器
- `docer rm <id|name>` 删除对应的容器
- `docker container prune` 删除无用的容器，以节省空间

注意：用 docker 运行的容器，默认不会持久化，容器被删掉，数据也就没了

## Docker 连接 MySQL

1. 进入 Docker 容器，相当于一个 Linux 虚拟机：
```bash
docker exec -it mysql-demo1 bash
```
2. 使用 MySQL 命令进入数据库
```bash
# -u root：以 ROOT 用户进入
# -p 直接回车，然后输入密码
mysql -u root -p
```
3. 使用 SQL 语句操作
```sql
# 查看有哪些数据库
show databases;
# 进入其中一个数据库
use <数据库名称>;
# 看有哪些表
show tables;
# 查看表内容
select * from <表名称>;
```

# 数据库的基本知识

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

# 用 Node.js 连接数据库

可以使用 [mysqljs 库](https://github.com/mysqljs/mysql)

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

# 常见的 SQL 命令

## 操作 database/table

- 增：`CREATE DATABASE IF NOT EXISTS harvey DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;` （MySQL 才需要使用 utf8mb4，并且在后面写 COLLATE）
- 删：`DROP DATABSE;`

## 操作记录

- 增：`INSERT INTO user (name, age) VALUES ('harvey', 20);`
- 删：`DELETE FROM user WHERE name='harvey';`
- 改：`UPDATE user SET age=24 WHERE name='harvey';`
- 查：`SELECT * FROM user LIMIT 10;`

# MySQL 的数据类型

五大类：数字、字符串、时间和日期、JSON（5.7.8 以上）、其他特殊类型

## 数字类型

- bit、tinyint、bool/boolean、smallint、mediumint、int、bigint、decimal、float、double

- serial：一个自增长的非负大整数（BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE）

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)

## 字符串类型

- char(100)、varchar(100)、binary(1024)、varbinary(1024)、blob、text、enum('v1','v2')、set('v1','v2')

varchar 长度可变，可以节省空间

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html)

## 时间和日期

- date、time、datetime、timestamp、year

[官方文档](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-type-syntax.html)

一般传给前端需要转换为 ISO 8601 格式

# Sequelize.js