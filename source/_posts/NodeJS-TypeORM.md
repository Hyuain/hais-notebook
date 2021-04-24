---
title: TypeORM
date: 2021-04-11 18:51:00
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

TypeORM 是一个 Node 中对 TypeScript 支持比较好的关系对象映射，支持关联、事务、数据库迁移，同类产品还有 Sequelize。

<!-- more -->

# 启动 Postgresql 数据库

在项目目录里面新增 `blog-data` 目录，并将其添加进 `.gitignore` 中。
 
然后启动 postgresql：

```bash
docker run -v "$PWD/blog-data":/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=blog -e POSTGRES_HOST_AUTH_METHOD=trust -d postgres
```

将上述命令的 `-e POSTGRES_HOST_AUTH_METHOD=trust` 替换成 `-e POSTGRES_PASSWORD=123456` 就可以设置密码

然后可以这样进入 postgresql：

```bash
docker exec -it id bash
psql -U username -W
```

然后可以执行 psql 的命令：

```bash
\l # list databases
\c # connect to a databases
\dt # display tables
```

# 创建 database

由于 TypeORM 没有为我们提供单纯创建数据库的 API，我们需要进入数据库用 SQL 语句来进行创建：

```postgresql
CREATE DATABASE xxx ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
```

一般来说需要创建三个数据库：development、test、production

# 安装 TypeORM

[官方文档在这里](https://typeorm.io/#/)。

注意！当按照官方文档操作，运行命令 `typeorm init --database postgres` 之后，`.gitignore` 、 `package.json` 和 `tsconfig.json` 将会被更改，这时需要检查他的更改并酌情处理！

比如当我们在使用 Next 搭配 TypeORM 的时候，会遇到问题是 Next 使用 Babel 处理 TypeScript，而 TypeORM 会安装 ts-node 依赖，这样就冲突了：
1. 先从 `package.json` 中删除命令擅自添加的 `ts-node`
2. TypeORM 想要我们直接运行 `src/index.ts`，但是没有 `ts-node` 我们应该如何运行这个文件呢？

