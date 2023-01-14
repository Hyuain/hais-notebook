---
title: Git & GitHub
date: 2020-01-29 20:58:04
tags:
  - 入门
  - 踩坑
categories:
  - [工具]
---

主要记录 Git 的基本用法及一些坑的解决。

<!-- more -->

# Git 本地仓库

## 六行配置

```bash
git config --global user.name [username] 
git config --global user.email [useremail@example.com]
git config --global push.default simple
git config --global core.quotepath false
git config --global core.editor "code --wait"
git config --global core.autocrlf input
```

## 命令

| 命令 | 含义 |
| --- | --- |
| `git init` | 创建 `.git` 目录，容纳代码快照 |
| `git add` | 准备将文件提交进 git 目录（本地仓库），路径可以是绝对路径、相对路径、`.`（当前目录）和 `*` |
| `git status` |  查看目前的状态 |
| `git commit` | 提交（实际上是复制到了.git目录） |
| `git commit -m "version1"` | 查看更新时间 |
| `git commit -v（--verbose）` | 推荐，提交理由写得更详细 |
| `git reset --hard xxxxxx（提交号的前六位）` | 这个操作会使没有 commit 过的变动消失 |
| `git log` | 查看历史记录 |
| `git reflog` | 查看包括 reset 的历史记录 |
| `git branch xxx` | 基于当前的commit创建一份新的分支 |
| `git branch` | 查看所有分支和当前分支 |
| `git branch -d xxx` | 删除分支 |
| `git checkout xxx` | 切换分支，当前未提交的文件如果与另一个分支不冲突，切换分支的操作就不会产生影响，如果冲突，可以用git stash或合并冲突 |
| `git merge xxx` | 合并分支，先到达想要保留的分支再使用 |
| `git rm --chached xxx` | 将已经被 add 的文件从缓存中删除 |

`.gitignore` 文件中可以输路径来忽略不想提交的文件，比如 `node_modules` `DS_Store` `.idea` `.vscode`

# Git 远程仓库

```bash
# 创建一个私钥和公钥
ssh-keygen -t rsa -b 4096 -C [useremail@example.com]
cat ~/.ssh/id_rsa.pub

# 测试
ssh -T git@github.com

# 上传一个本地仓库
git remote add origin [git@github.com:Hyuain/git-demo-1.git]
git push -u origin master # 一个新的分支需要写 -u 和后面的

# 下载一个远程仓库
git clone
# git clone git@?/xxx.git：创建一个新目录，与 xxx 同名
# git clone git@?/xxx.git yyy：创建一个新目录，命名为 yyy
# git clone git@?/xxx.git .：不会新建目录，使用当前目录容纳代码和 .git

# 其他
# git stash：把文件藏起来
# git stash pop：再把文件取出来
```

## 高级操作

```bash
touch ~/.bashrc
echo 'alias ga="git add"'>> ~/.bashrc
echo 'alias gc="git commit -v"'>> ~/.bashrc
echo 'alias gl="git pull"'>> ~/.bashrc
echo 'alias gp="git push"'>> ~/.bashrc
echo 'alias gco="git checkout"'>> ~/.bashrc
echo 'alias gst="git status -sb"'>> ~/.bashrc

code ~/.bashrc
# 在文件最后加上
alias glog="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit -- | less"
```

# 踩坑记录

## git log 中文乱码

输入以下命令即可解决

```bash
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
export LESSCHARSET=utf-8
```

## git push / clone 报错

```bash
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

### 方法 1

防火墙添加 22 端口入站策略配置为允许，有时候不能解决问题。

### 方法 2

1. 在 `~/.ssh` 存放密钥（`id_rsa` 和 `id_rsa.pub`）的文件夹，新建 `config`，内容如下：

```text
Host github.com
User XXX@xx.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

2. 刷新 `config`

```bash
git config --global user.name "XXX"
git config --global user.email XXX@xx.com
```

3. 测试

```bash
ssh -T git@github.com
```

有时候还是不能解决问题

### 方法 3

1. 修改 hosts 文件，也可以进入 [IPAddress](https://www.ipaddress.com/) 查询这域名的 IP 地址。

```text
192.30.253.112 github.com
185.199.108.153 assets-cdn.github.com
185.199.109.153 assets-cdn.github.com
185.199.110.153 assets-cdn.github.com
185.199.111.153 assets-cdn.github.com
199.232.5.194 github.global.ssl.fastly.net
```

2. 刷新 DNS 缓存

```cmd
ipconfig /flushdns
```

# GitHub 加速

## 推荐 - 通过 `SSH` 时

`SSH` 的clone方式也是大家使用的最多的方式，但 `SSH` 一般情况是不走 `http`、`https` 的代理的，在终端执行 `export https_proxy=xxx`，甚至将代理软件设置为全局代理也无法令 `SSH` 通过代理。这里介绍如何使`SSH clone`时走代理：
把下面这段写进 `~/.ssh/config` 文件（因为用到私玥认证所以带了 `IdentityFile` 选项）

```bash
# linux & WSL 配置
Host github.com
    User git
    ProxyCommand /usr/bin/nc -x 127.0.0.1:7890 %h %p
    IdentityFile ~/.ssh/id_rsa
```

```bash
# GitBash 的配置
 Host github.com
     User git
     ProxyCommand connect -H 127.0.0.1:7890 %h %p
     IdentityFile ~/.ssh/id_rsa
```

这里 `127.0.0.1:7890` 是一个 `http` 代理，这样便能令 `SSH` clone GitHub 时通过代理，达到加速的目的。

这种设置方法依赖Linux命令[`nc`](https://linux.die.net/man/1/nc)和 [`ssh config`](https://linux.die.net/man/5/ssh_config) `ProxyCommand`。

> - 使用时可能会有的报错：`Bad owner or permissions on /Users/username/.ssh/config`，解决方法参考如下：
    >
    >   https://github.com/ddollar/heroku-accounts/issues/15
    >
    >   https://docs.github.com/en/github/authenticating-to-github/error-permission-denied-publickey

## 通过 `HTTPS` 时

通过 `HTTPS` 形式的 clone 通过代理加速很简单，和普通的网页访问的加速并没有不一样：

```
export HTTPS_PROXY=socks5://127.0.0.1:1080
git clone https://github.com/lightjiao/lightjiao.github.io.git
```

另外，`Git` 本身有一个 `http.proxy/https.proxy` 的设置：

```
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

但 `HTTPS` clone的方式本身有一个缺点，每次与远端repo通讯时候都会要求填写用`Github`户名和密码，使用比较繁琐

## A Little More on Git Protocol

By saying "Git Protocol", I mean a Git Server Protocal called "The Git Protocol". Its URI likes `git://server/project.git`. Note it starts with a `git://`, not `ssh://` or `http://` or something else.

It's not commonly used, so you can skip this. I write this mainly for completeness of Git proxy settings.

Git a has configration `core.gitProxy` dedicated for this protocol, its man reads:

```
core.gitProxy

A "proxy command" to execute (as command host port) instead of establishing direct connection to the 
remote server when using the Git protocol for fetching. If the variable value is in the 
"COMMAND for DOMAIN" format, the command is applied only on hostnames ending with the specified 
domain string. This variable may be set multiple times and is matched in the given order; the first 
match wins.

Can be overridden by the GIT_PROXY_COMMAND environment variable (which always applies universally, 
without the special "for" handling).
```

You can set it by `git config`.

参考来源：

- https://www.zhihu.com/question/27159393/answer/145122287
- https://gist.github.com/coin8086/7228b177221f6db913933021ac33bb92
