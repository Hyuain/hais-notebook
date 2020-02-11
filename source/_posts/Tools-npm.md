---
title: npm & yarn
date: 2020-02-02 15:34:21
tags:
  - 入门
  - 效率
  - 收集
  - 踩坑
categories:
  - [工具]
---

收集 npm 的包和问题。

<!-- more -->

# 基础配置

```bash
npm i -g nrm
nrm use taobao
# 所有的全局安装都在 C:\Users\Zhang\AppData\Roaming\npm\
```

```bash
yarn global add yrm
yrm use taobao
# 用 yrm ls 可以看所有源
```

# 一些问题

## 怎么知道包的最新版是多少？

```bash
npm info [name] version
```

