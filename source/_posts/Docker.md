---
title: Docker
date: 2021-03-14 10:49:35
tags:
  - 入门
categories:
  - [工具]
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
- `docker rm <id|name>` 删除对应的容器
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
