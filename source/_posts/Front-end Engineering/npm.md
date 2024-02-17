---
title: npm & yarn
date: 2020-02-02 15:34:21
categories:
  - - 工具
tags:
  - F1
---

npm 是 [[NodeJS]] 的包管理器。

本文收集 npm 的包和问题。

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

[文档在这里](https://docs.npmjs.com/cli/v7/configuring-npm/package-lock-json)

对于 npm 来说，`package.json` 可以看做他的输入，`node_modules` 可以看成他的输出。

那么在理想情况下 npm 应该是一个 **纯函数**，但大多数时候并不能做到这一点，主要是因为：

- 使用者 npm 版本可能不同，不同版本有不同的算法
- `^2.6.1` 这样的写法会使得安装的版本不同
- 我们可以写死版本，但却无法控制包的依赖的升级

为了解决这个问题，npm5.x 开始加入了 `package-lock.json` 文件，每当 `npm install` 的时候，npm 都会产生/更新 `package-lock.json` 文件，他的作用就是锁定当前依赖安装结构。

我们需要注意 `pacakge-lock.json` 中的 `requires` 和 `dependencies` 两个属性的不同：

- `requires` 表示包的依赖，与包的 `package.json` 中的 `dependencies` 依赖项相同
- `dependencies` 是存在包的 `node_modules` 中的依赖（不是所有的包都有，只有当包的依赖包的版本与根目录 `node_moduels` 中的依赖冲突时才会有

从 npm3.x 开始，就采用扁平化的方式安装 `node_modules`。在安装的时候，npm 会遍历整个依赖树，不管是项目的直接依赖还是依赖的依赖，都会优先安装在根目录的 `node_modules` 中。遇到相同名称的包，如果发现根目录的 `node_modules` 中存在，但不符合 semver-range，就会将包的依赖装在包的 `node_modules` 中。

## npm 安装的流程

1. 从磁盘加载 `node_modules` 树
2. 克隆 `node_modules` 树
3. 获取 `package.json` 文件和分类完毕的元数据信息，并把元数据信息插入到克隆树中
4. 遍历克隆树，检查是否有丢失的依赖。如果有，把他们添加到克隆树中，依赖会尽可能地添加到最高层
5. 比较原始树和克隆树，列出将原始树转换为克隆树需要采取的具体步骤
6. 执行具体步骤，包括 install、update、remove、move 等

### 例1

假设 `package{dep}` 结构代表包和包的依赖，现在有 `A{B,C}` `B{C}` `C{D}`，按照算法执行完毕之后，生成 `node_modules` 如下：

```text
A
+-- B
+-- C
+-- D
```

在安装 `B` 的依赖——`C` 的时候，发现顶层已经有 `C` 了，于是不会再在 `B` 的 `node_modules` 中再次安装；安装 `D` 的时候发现顶层没有 `D`，于是就将 `D` 安装在顶层。

### 例2

当我们使用 `A{B,C}` `B{C, D@1}` `C{D@2}` 这样的依赖关系，产生的结果如下：

```text
A
+-- B
+-- C
    +-- D@2
+-- D@1
```

`B` 依赖 `D@1`，安装时发现顶层没有，就会把 `D@1` 装在顶层；`C` 又依赖了 `D@2`，但安装 `D@2` 的时候发现顶层已经有了，npm 不允许同层存在两个名字相同的包，于是就将 `D@2` 装在了 `C` 自己的 `node_modules` 中。

模块安装的顺序会影响当存在相同依赖时，哪个版本会被安装在顶层。首先是项目中依赖的第一层包，然后这些包会被按照 a-z 的顺序依次进行安装（与在 `package.json` 中写入的顺序无关）。

### 例3

有时，我们项目中所引用的包版本比较低，比如 `A{B@1, C}`，但 `C{B@2}`，首先安装的结构如下：

```text
A
+-- B@1
+-- C
    +-- B@2
```

然后我们将项目中的 `B` 升级到 `B@2`，理想结构应该如下：

```text
A
+-- B@2
+-- C
```

但现在 `package-lock.json` 却是这样的：

```text
A
+-- B@2
+-- C
    +-- B@2
```

`B@2` 出现了两份，一份在顶层，一份在 `C` 中，这个时候需要我们手动执行 `npm dedupe` 进行去重操作。

## npm ci

现在我们知道了，`package-lock.json` 在每次 `npm install` 的时候都可能发生更新，他只是精确描述了当前安装的 `node-moduels` 树。并不是说 `package-lock.json` 中是什么版本，我用 `npm install` 安装的就是什么版本——若想要达成这样的效果，需要使用 [`npm ci`](https://docs.npmjs.com/cli/v7/commands/npm-ci)

ci 就是 `clean install` 的意思，他不会改变 `package-lock.json`，并且会完全按照 `package-lock.json` 中的样子进行安装，比较适合用于一些自动生成的环境，比如测试平台、持续集成等。

# npm-shrinkwrap.json

在 npm5.x 之前，可以手动通过 `npm srhinkwrap` 生成 `npm-shrinkwrap.json` 文件，他与 `package-lock.json` 作用相同。当二者在项目中同时存在的时候，将优先使用 `npm-shrinkwrap.json`

# `npm-outdated`

可以查看当前的依赖的包版本是否过时，默认只列出顶层，可以通过 `--depth` 参数控制深度。

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
