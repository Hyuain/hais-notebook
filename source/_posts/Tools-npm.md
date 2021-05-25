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
参考了 [1](https://segmentfault.com/a/1190000039289332) , [2](https://juejin.cn/post/6844904160001785869)

<!-- more -->

# 依赖包分类

## dependencies 与 devDependencies

| dependencies | devDependencies |
| --- | --- |
| `npm install [name]` | `npm install [name] --save-dev`
| 生产环境，属于线上代码的一部分 | 开发时才需要的依赖 |
| vue、element-ui 等 | webpack、babel-loader 等 |

- `npm install` 的时候，两个模块的包都会被下载
- 可以通过 `npm install --production` 忽略开发依赖，只安装基本依赖
- 模块是否被打包，取决于项目是否引入了该模块，而不取决于是在 dependencies 还是在 devDependencies 中
- 对于普通项目来说，把包写在哪里区别不大，但如果将这个包发布，那么写在 devDependencies 中的依赖将不会被下载

# 一些问题

## 怎么知道包的最新版是多少？

```bash
npm info [name] version
```

# 好用的 npm 包

## nrm

一个 npm 镜像管理工具。

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
