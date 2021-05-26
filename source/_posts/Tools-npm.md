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
参考了 [1](https://segmentfault.com/a/1190000039289332) , [2](https://juejin.cn/post/6844903870578032647)

<!-- more -->

# 依赖包分类

## dependencies, devDependencies

| dependencies | devDependencies |
| --- | --- |
| `npm install [name]` | `npm install [name] --save-dev`
| 生产环境，属于线上代码的一部分 | 开发时才需要的依赖 |
| vue、element-ui 等 | webpack、babel-loader 等 |

- `npm install` 的时候，两个模块的包都会被下载
- 可以通过 `npm install --production` 忽略开发依赖，只安装基本依赖
- 模块是否被打包，取决于项目是否引入了该模块，而不取决于是在 dependencies 还是在 devDependencies 中
- 对于普通项目来说，把包写在哪里区别不大，但如果将这个包发布，那么写在 devDependencies 中的依赖将不会被下载

## peerDependencies, bundleDependencies, optionalDependencies

这三个属性主要是面向包的发布者。

### peerDependencies

表明如果想要使用这个包，需要的宿主环境所安装的包。

比如在 `vuei` 的 `1.0.0-alpha.24` 版本中，有：

```json
{
  "peerDependencies": {
    "vue": "^2.5.16"
  }
}
```

表明如果在要使用该版本的 `vuei`，需要 `vue` 的版本 `>=2.5.16` 且 `<3.0.0`。

在 npm3.x 以上的版本中，如果安装结束后宿主环境没有满足要求，就会在控制台打出警告。

### bundledDependencies

如果想在本地保留一个完整的 npm 包或者想生成一个压缩文件来获取 npm 包，会用到这个 bundleDependencies。使用 `npm pack` 进行打包的时候会将 bundleDependencies 中的包一起打包，并在 `npm install` 的时候进行安装。

比如：

```json
{
  "name": "my-npm-pack",
  "version": "1.0.0",
  "bundleDependencies": [
    "renderized",
    "super-streams"
  ]
}
```

当我们执行 `npm pack` 之后会生成  `my-npm-pack-1.0.0.tgz` 文件，该文件中包含 `renderized` 和 `super-streams` 依赖。当执行 `npm install my-npm-pack-1.0.0.tgz` 时，会自动安装这两个依赖。他们的版本需要在 dependencies 中指定。

当使用 `npm publish` 来发布包的话，这个属性将不起作用。

### optionalDependencies

写在 optionalDependencies 里面的包，如果安装失败了也不会影响安装过程，并且 optionalDependencies 中的配置会覆盖 dependencies 中对应包的配置。

# scripts

- 定义在 `scripts` 中的命令，可以用过 `npm run <command>` 来执行
- `test` `start` `restart` `stop` 可以不用加 `run`
- `npm run env` 可以获取到脚本运行时所有的环境变量
- `npm run` 会将 `node_modules/.bin/` 加入到 `shell` 的环境变量 `PATH` 中，这样局部安装的包就可以直接执行而不用加 `node_modules/.bin/` 的前缀

# package-lock.json



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
