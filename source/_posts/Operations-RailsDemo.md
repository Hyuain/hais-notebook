---
title: 部署到阿里云：以 Rails App 为例
date: 2021-03-26 22:59:12
tags:
  - 入门
categories:
  - [全栈, 运维]
---

一个部署的文档：阿里云、Docker、数据库、Rails 镜像、自动化。

<!-- more -->

# 购买并登录阿里云服务器

1. 去阿里云选择 ECS 服务器，购买一个。登录凭证选择使用密钥对。
2. 使用 `ssh root@公网ip` 登录到机器上
  1. 公网 ip 很难记，建议记到 hosts 文件里

# 创建低权限用户

1. 进入 root 用户
2. 安装 git：`apt update; apt install git -y`
3. 创建 harvey 用户：`adduser harvey`，密码为 `123456`
4. 让 harvey 用户也能 ssh 登录
  1. `mkdir -p /home/harvey/.ssh`
  2. `cp ~/.ssh/authorized_keys /home/harvey/.ssh/`
  3. `chown -R harvey:harvey /home/harvey/.ssh`
5. 退出 root（使用 `exit` 或 `ctrl + D`）
6. 现在就可以使用 `ssh root@harvey` 登录 harvey 用户了

# 安装 Docker

1. 进入 root 用户
2. 根据 [菜鸟教程](https://www.runoob.com/docker/ubuntu-docker-install.html) 来安装 Docker
3. 将 harvey 添加到 docker 用户组：`usermod -aG docker harvey`，这样 harvey 才能运行 docker 命令
4. 运行 `docker run hello-world`
5. 如果 `docker run hello-wrold` 运行很慢，可以参考 [这篇文章](https://developer.aliyun.com/article/29941) ，在 [这里](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) 配置镜像加速器。

# 用 Docker 启动数据库

1. 进入 harvey 用户
2. 创建数据库目录 `mkdir /home/harvey/data`
3. 开启数据库 `docker run --net=host -v /home/harvey/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=harvey -e POSTGRES_PASSWORD=123456 --name psql1 -d postgres`
  1. 注意如果 `-net=host` 没有写的话会导致两个容器网不通
  2. 此处用了端口 5432 而不是 5001，因为 `-net=host` 不支持 `5001:5432`
4. 测试数据库
  1. `docker exec -t psql1 bash` 进入容器
  2. `psql -U harvey`
  3. `\l` 列出所有可用数据库

# 测试 Ruby 容器

1. 进入 harvey 用户
2. 创建测试文件 `echo "p 'Hi, I am Harvey'" > test.rb`
3. 使用 ruby 容器运行测试文件：`docker run -it --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp ruby ruby test.rb`
4. 看到 `Hi, I am Harvey` 就说明测试成功了，然后用 `rm test.rb` 删除测试代码

# 本地构建 Rails 镜像

1. 创建 `Dockerfile`：
```dockerfile
# 从哪里下载镜像，可以指定目录
FROM ruby

# 工作目录
WORKDIR /usr/src/app
# 把源代码拷贝到这个镜像中
COPY Gemfile .
COPY Gemfile.lock .
# 在镜像中运行
RUN gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
RUN gem install bundler
RUN bundle config mirror.https://rubygems.org https://gems.ruby-china.com
RUN bundle install
# 将所有的文件都拷贝到这个镜像中
COPY . .
# 更新 bin 目录
RUN bundle exec rake app:update:bin
# 暴露 3000 短开口
EXPOSE 3000
# 入口命令
CMD [ "bin/rails", "server", "-b","0.0.0.0", "-p","3000"]
```
2. 将 `config/database.yml` 复制到 `config/database.sample.yml`，再把原来的 `config/database.yml` ignore 掉
3. `docker build -t harvey/rails-demo:0.1 .`，得到镜像 `harvey/rails-demo` 版本为 0.1
3. 运行刚刚得到的镜像：`docker run --net=host -p 3000:3000 harvey/rails-demo:0.1`