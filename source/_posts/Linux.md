---
title: Linux
date: 2022-05-29 16:25:00
tags:
  - 入门
  - 运维
categories:
  - [全栈, 运维]
---

参考文章 [1](https://juejin.cn/post/6917096816118857736)、B 站教程等。

<!-- more -->

# Linux 发行版

## Linux 历史

- 1983 年，Richard Stallman 发起 GNU 计划，他同时也是 smalltack 语言的发明者（被公认的第二个面向对象语言），他还编写了 Emacs。
- 1991 年，Linus Torvalds 开始开发 Linux 内核。
- 1996 年，确定企鹅为 Linux 的吉祥物。

## Linux 发行版的主要起源

Linux 有众多发行版，但其实有两条主线，**debian** 系列和 **redhat** 系列。

- Debian 计划是一个致力于创建自有操作系统的合作组织，他的特点是稳定、安全。Ubuntu 就是基于 Debian 改进的。
- RedHat 是一家商业公司，现在对个人提供一些*红帽认证*之类的证书。现在的 Centos 占据了大部分服务器市场。

## 典型版本

- Ubuntu：基于 Debian 系统的 unstable 分支修改的，界面漂亮，包管理软件是 apt-get。
- Centos：是 RHEL 源码再编译的产物，在包管理和稳定性上，与红帽企业版差别不大。
- ArchLinux：采用滚动升级的模式发行，他的软件和理念通常都是最新的，定制化非常强。
- Gentoo：下载的是软件的源代码，然后在本地进行编译安装，比较底层。
- LFS：Linux From Scratch，有非常详细的安装文档，教你怎么编译内核，编译引导程序等。
- Kali：非常专业的发行版，包含了常见的破解、渗透、攻击工具。

# PATH

Linux 上一些目录里的文件，是可以被默认找到的，这些目录的集合，就叫做 PATH。PATH 也是一个环境变量，我们可以额通过命令查看 PATH：

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

# Linux 一切皆文件

在 Linux 中，对命令、文档、设备、目录、套接字的操作都是一致的。开发驱动程序中使用的一些函数，其实和读写文件没什么两样。

## 当前路径：`pwd`

> pwd: print working directory

{% note warning %}
在执行一些危险命令时，时常确认当前目录是是个好习惯。
{% endnote %}

```bash
pwd
# /root
```

当默认使用 root 登录之后，就会停留在 `/root` 目录中，Linux 中的目录层次，是通过 `/` 进行划分的。

## 文件系统的用户标准

> FHS: Filesystem Hierarchy Standard

| 第一层 | 第二层 | 介绍 |
| --- | --- | --- |
| `/bin` | | 目录 `/usr/bin` 的软链接 |
| `/sbin` | | 目录 `/usr/sbin` 的软链接 |
| `/lib` | | 目录 `/usr/lib` 的软链接 |
| `/usr` | `/bin` | 存放一些常用的命令 |
| `/usr` | `/sbin` | 存放一些管理员常用的命令 |
| `/usr` | `/lib` | 存放一些动态库和模块文件 |
| `/sys` | | 内核中的数据结构的可视化接口 |
| `/proc` | | 内存映像 |
| `/run` | | 内存映像 |
| `/boot` | | 存放引导程序，内核相关文件 |
| `/dev` | | 存放一些设备文件，比如光盘 |
| `/etc` | | 存放一些全局的、应用配置文件。比如如果安装了 php、nginx，他们的配置文件就放在这下面的某个文件夹里面 |
| `/var` | | 同 `/var/run`，存放系统运行时需要的文件，比如 mysql 的 pid 等 |
| `/tmp` | | 非常特殊的临时文件夹，断电丢失，所有用户都有写入权限，通常用来做文件交换用 |
| `/home` | `/*` | 用户目录。我们可以在里面做任何操作，可以存放一些自己的资料、测试用的数据资料等 |
| `/root` | | root 用户的 home |

## 查看文件列表：`ls`

```bash
# ls 可以接受路径参数，不用跳转到那里去就能输出相关信息
ls /
```

| 常用参数 | 解释 |
| --- | --- |
| `-a` `--all` | 列出所有的文件，包括隐藏文件 |
| `-l` | 列出文件的详细信息 |
| `-t` | 根据修改时间排序 |
| `-c` | 根据 ctime 排序显示 |
| `-C` `--color` | 显示彩色 |

```text
lrwxrwxrwx   1 root root         7 Apr 20 18:39 bin -> usr/bin
drwxr-xr-x   3 root root      4096 Apr 20 18:54 boot
drwxr-xr-x  18 root root      3880 May 29 16:48 dev
drwxr-xr-x  88 root root      4096 May 23 17:14 etc
drwxr-xr-x   3 root root      4096 May 23 17:03 home
lrwxrwxrwx   1 root root         7 Apr 20 18:39 lib -> usr/lib
```

- 第一列：第一个字母是文件类型，后面表示文件权限
- 第二列：普通文件表示链接数，目录表示是子文件数
- 第三列：所属用户
- 第四列：所属组
- 第五列：大小
- 第六列：最后修改时间
- 最后一列：软链接或文件名

### 隐藏文件

直接在 `/root` 目录里执行 `ls -al` 可以看到更多的东西，这些额外的东西就是隐藏文件，他们都是以 `.` 开头，并且以配置文件居多。

```text
drwx------  5 root root 4096 May 29 16:55  .
drwxr-xr-x 18 root root 4096 May 23 16:52  ..
-rw-------  1 root root 1196 May 29 19:45  .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019  .bashrc
```

并且还有两个特殊的目录—— `.` 和 `..`。前者表示当前目录，后者表示上层目录。

也可以使用 `ll`，他相当于 `ls -al`。

## 切换目录：`cd`

| 常用参数 | 解释 |
| --- | --- |
| `../` | 上层目录 |
| `./` | 当前目录 |
| `~` | 当前的用户目录 |
| `-` | 可以在最近的两次目录中来回切换 |

## 创建目录：`mkdir`

```bash
mkdir -p a1/b2/c3/d4/e5/f6/{g7,g8,g9,g10}
```

`-p` 可以让我们一次性创建多层目录，`{}` 可以一次性创建多个目录

## 文件操作：`touch` `rm` `cp` `mv`

```bash
# 创建文件
touch long.txt
touch haha.txt

# 复制文件
cp haha.txt xixi.txt

# 移动文件
mv long.txt short.txt
mv haha.txt a1/
mv xixi.txt a1/b2

# 删除文件
rm short.txt
rm -rvf a1/
```

### 复制文件：`cp`

| 常用参数 | 解释 |
| --- | --- |
| `-r` | 递归复制 |
| `-f` | 强制复制，直接覆盖不给提示 |
| `-i` | 覆盖时给提示 |
| `-v` | 显示详细步骤 |
| `-p` | 把文件属性也复制过去 |
| `-a` | 把档案所有特性一起复制 |

## 删除文件：`rm`

| 常用参数 | 解释 |
| --- | --- |
| `-r` | 递归删除 |
| `-f` | 强制删除，不给提示 |
| `-i` | 删除时给提示 |
| `-v` | 显示详细步骤 |

## 移动文件：`mv`

| 常用参数 | 解释 |
| --- | --- |
| `-f` | 直接覆盖，不给提示 |
| `-i` | 覆盖时给提示 |
| `-v` | 显示详细步骤 |
| `-b` | 覆盖前先备份 |
| `-u` | 文件较新的时候才覆盖 |

# 操作文件

`ll` 结果的第一列的第一个字母，表示了 Linux 文件类型：

| 字母 | 解释 |
| --- | --- |
| `-` | 普通文件 |
| `d` | 目录文件 |
| `l` | 链接文件，比如快捷方式 |
| `s` | 套接字文件 |
| `c` | 字符设备文件，比如 `/dev/ 中的很多文件` |
| `b` | 块设备文件，比如一块磁盘 |
| `p` | 管道文件 |

Linux 上文件可以没有后缀，他的后缀其实没有什么实际的意义，我们可以通过 `file` 命令查看文件的具体类型：

```bash
file /etc
# /etc: directory
file /etc/group
# /etc/group: ASCII text
```

## 创建一个文件

### 生成数字序列：`seq`

```bash
seq 10 20 >> spring
```

`>` 是将前面命令的输出重定向到其他地方；`>>` 类似，并且是在原来文件的基础上追加内容

### 查看内容：`cat`

```bash
cat -n spring
# 1	10
# 2	11
# 3	12
# 4	13
# 5	14
# 6	15
# 7	16
# 8	17
# 9	18
# 10	19
# 11	20
```

`-n` 会显示行号。

```bash
# 合并 a 文件和 b 文件到 c 文件
cat a b >> c

# 把 a 文件内容作为输入，使用管道处理
cat a | cmd

# 写入内容到指定文件
cat > index.html <<EOF
<html>
  <head><title></title></head>
</html>
EOF
```

### 平和地查看内容：`less` `head` `tail`

如果文件非常大，使用 `cat` 就会在终端上疯狂输出，这时候我们可以使用另外的方案。

#### `less`

| 常用参数 | 解释 |
| --- | --- |
| `-n` | 显示行号 |
| `-m` | 显示百分比 |

| 常用快捷键 | 解释 |
| --- | --- |
| `/` | 向下搜索 |
| `?` | 向上搜索 |
| `n` | 重复上一个搜索 |
| `N` | 反向重复上一个搜索 |
| `<space>` | 向后翻页 |
| `b` | 向前翻页 |
| `j` | 向下一行 |
| `k` | 向上一行 |
| `g` | 去开头 |
| `G` | 去结尾 |
| `q` | 退出 |

#### `head` `tail`

```bash
# 显示头三行
head -n 3 spring

# 显示尾三行
tail -n 3 spring
```

`tail -f` 表示循环读取，可以用来在控制终端实时监控文件变化，来查看一些滚动日志，可以配合 `grep` 命令达到过滤效果

```bash
# 滚动查看系统日志
tail -f /var/log/messages

# 滚动查看包含 info 字样的日志信息
tail -f /var/log/messages | grep info

# 滚动监控到重新创建的文件，比如一些按天滚动的
tail -F
```

### 查找文件：`find`

| 常用参数 | 解释 |
| --- | --- |
| `-name` | 文件名，可以使用通配符 |
| `-size` | 根据大小搜索 |
| `-type` | 根据文件类型搜索 f d l s p b c |
| `-perm` | 根据文件权限搜索 |
| `-mtime` | 修改时间，天 |
| `-mmin` | 修改时间，分 |
| `-user` | 属于某个用户 |
| `-group` | 属于某个组 |
| `-exec` | 执行命令，建议使用 xargs 替代 |

```bash
# 全局找 decorator 文件
find / -name decorator.py -type f

# 删除当前目录中的所有 class 文件
find . | grep .class$ | xargs rm -rvf

# 找到 /root 下一天前访问的文件
find /root -atime 1 -type f

# 查找十分钟内更新过的文件
find /root -cmin -10

# 查找输入 root 用户的文件
find /root -user root

# 查找大于 1MB 的文件，进行清理

find /root -size +1024k -type f | xargs rm -f
```

 