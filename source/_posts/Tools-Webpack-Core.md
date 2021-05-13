---
title: Webpack 核心原理
date: 2021-03-03 22:47:21
tags:
  - 入门
categories:
  - [工具, Webpack]
---

我们可以通过 Babel 提供的 Parser、Traverse、Generator 转换代码、分析依赖。
Webpack 的核心就是通过 Babel 将 ESModule 的语法转变为 CommonJS，使得浏览器支持，并将所有的文件打包成一个 js。

<!-- more -->

# Babel

## 原理

1. **Parse**(`@babel/parser`): 把代码变成 AST
2. **Traverse**(`@babel/traverse`)：遍历 AST 进行修改，或者遍历 AST 找到源代码的依赖
3. **Generate**(`@babel/generator`)：把新的 AST 变成另外的代码

`@babel/core` 包含了前三者，`@babel/preset-env` 中有很多内置的规则

> 为什么要用 AST？
> 因为显然用正则表达式是不太科学的，需要识别到每个单词的意思

## 示例：把 let 变成 var

```typescript
import { parse } from "@babel/parser"
import traverse from "@babel/traverse"
import generate from "@babel/generator"

const code = `let a = "let"; let b = 2`
const ast = parse(code, { sourceType: "module" })
traverse(ast, {
  // 进入时执行的钩子
  enter: item => {
    const { type, kind } = item.node
    if (type === "VariableDeclaration" && kind === "let") {
      item.node.kind = "var"
    }
  }
})
const result = generate(ast, {}, code)
console.log(result.code)
```

可使用命令执行上述 DEMO：

```bash
node -r --inspect-brk ts-node/register [filename]
```

## 示例：把代码转变为 ES5

```typescript
import { parse } from "@babel/parser"
import * as babel from "@babel/core"

const code = `let a = "let"; const b = 2`
const ast = parse(code, { sourceType: "module" })
const result = babel.transformFromAstSync(ast, code, {
  presets: ["@babel/preset-env"]
})
```

## 示例：依赖分析

```typescript
traverse(ast, {
  enter: item => {
    if (item.node.type === "ImportDeclaration") {
      // import 后面的路径，往往是一个相对路径
      console.log(item.node.source.value)
    }
  }
})
```

1. 可以用一个 hash 表来存放依赖关系
2. 可以用递归实现嵌套依赖分析
3. 可以记住以前分析过的依赖，如果发现分析过就直接 return，这样就能实现对循环依赖的静态分析（不执行代码，只进行字面理解）

# Webpack 核心原理

## 让浏览器支持 import / export

根据 [MDN 上的描述](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)，在浏览器中，import 语句只能在声明了 type="module" 的 script 的标签中使用。并且 IE 是不支持这个特性的。

- 兼容策略 1：把代码全部放在 `<script type="module">` 中，这样会导致 IE 不兼容，并且会导致文件请求过多，因为每个被依赖的文件都需要被单独发出请求
- 兼容策略 2：把关键字转译为普通的代码，并且把所有的文件打包成一个文件

## @babel/core 帮助我们转义 import 和 export

babel 会帮助我们将 ESModule 的语法按照 CommonJS 的规则书写：import 关键字会变成 require 函数，export 关键字会变成 exports 对象

```typescript
import * as babel from "@babel/core"
const es5Code = babel.transform(code, {
  presets: ['@babel/preset-env']
})
```

```javascript
// 原始代码
import b from './b.js'
const a = {
  getB: () => b.value,
}
export default a
```

```javascript
// babel 转译之后的 es5 代码
// 使用严格模式
"use strict";

Object.defineProperty(exports, "__esModule", { value: true }); // 给当前模块添加 __esModule 属性，方便与 CommonJS 分开
exports["default"] = void 0;

var _b = _interopRequireDefault(require("./b.js"));

// 其实是为了给这个模块增加 default，因为 CommonJS 模块没有默认导出，这样是方便兼容，大部分 _interop 开头的代码都是为了兼容旧代码
function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { "default": obj };
};

var a = {
  getB: function getB() {
    return _b["default"].value;
  }
};
var _default = a;
exports["default"] = _default;
```

## 将所有文件打包成一个文件

> 这个文件**包含**了所有的模块，并且能**执行**所有的模块

思路：

```javascript
// 最终文件应该长这个样子
var depRelation = [
  { key: "index.js", deps: ["a.js", "b.js"], code: function(){ } },
  { key: "a.js", deps: ["c.js"], code: function(){ } },
  { key: "b.js", deps: ["d.js"], code: function(){ } },
] // 这样一个 depRelation 数组存放了依赖关系，他的第一项就是入口文件
// 通过 execute 执行第一个文件，这样就会自动执行依赖中的其他文件
excute(depRelation[0].key)
function execute(key) {
  var item = depRelation.find(i => i.key === key)
  item.code() // 我们要执行依赖中的代码，因此上面的 code 最好是一个函数
}
```

### 问题一：如何收集依赖？

这个问题在上面已经讨论过了，可以借助 AST 来分析依赖

### 问题二：如何将文件中的 code 变为一个函数方便我们执行？

我们可以这样将文件里面的 code 变为函数：

```javascript
code = `
  var b = require("./b.js")
  exports.default = "a"
`
// require、module、exports 是 commonJS 2 规范所定
code2 = `function(require, module, exports) {
  ${code}
}`
// 这样做最后写到文件中之后就是函数了
```

### 问题三：execute 函数应该怎么写？

下面是基本实现思路：

```javascript
const modules = {} // modules 用于缓存所有模块
function execute(key) {
  if (modules[key]) { return modules[key] }
  var item = depRelation.find(i => i.key === key)
  // require 其实就是 excute，区别在于 require 后面接的是一个路径，我们需要把它转成 key
  var require = (path) => {
    return execute(pathToKey(path))
  }
  modules[key] = { __esModule: true }
  var module = { exports: modules[key] }
  // 为了这个 item.code 在执行完成之后把自己的导出挂在 module.exports 上
  item.code(require, module, module.exports)
  return modules[key] // 其实就是 module.exports
}
```

## 一个简易的打包器

copy 过来的代码，仅供自己学习使用。

```typescript
// 请确保你的 Node 版本大于等于 14
// 请先运行 yarn 或 npm i 来安装依赖
// 然后使用 node -r ts-node/register 文件路径 来运行，
// 如果需要调试，可以加一个选项 --inspect-brk，再打开 Chrome 开发者工具，点击 Node 图标即可调试
import { parse } from "@babel/parser"
import traverse from "@babel/traverse"
import { writeFileSync, readFileSync } from 'fs'
import { resolve, relative, dirname } from 'path';
import * as babel from '@babel/core'

// 设置根目录
const projectRoot = resolve(__dirname, 'project_1')
// 类型声明
type DepRelation = { key: string, deps: string[], code: string }[]
// 初始化一个空的 depRelation，用于收集依赖
const depRelation: DepRelation = [] // 数组！

// 将入口文件的绝对路径传入函数，如 D:\demo\fixture_1\index.js
collectCodeAndDeps(resolve(projectRoot, 'index.js'))

writeFileSync('dist_3.js', generateCode())
console.log('done')

function generateCode() {
  let code = ''
  code += 'var depRelation = [' + depRelation.map(item => {
    const { key, deps, code } = item
    return `{
      key: ${JSON.stringify(key)}, 
      deps: ${JSON.stringify(deps)},
      code: function(require, module, exports){
        ${code}
      }
    }`
  }).join(',') + '];\n'
  code += 'var modules = {};\n'
  code += `execute(depRelation[0].key)\n`
  code += `
  function execute(key) {
    if (modules[key]) { return modules[key] }
    var item = depRelation.find(i => i.key === key)
    if (!item) { throw new Error(\`\${item} is not found\`) }
    var pathToKey = (path) => {
      var dirname = key.substring(0, key.lastIndexOf('/') + 1)
      var projectPath = (dirname + path).replace(\/\\.\\\/\/g, '').replace(\/\\\/\\\/\/, '/')
      return projectPath
    }
    var require = (path) => {
      return execute(pathToKey(path))
    }
    modules[key] = { __esModule: true }
    var module = { exports: modules[key] }
    item.code(require, module, module.exports)
    return modules[key]
  }
  `
  return code
}

function collectCodeAndDeps(filepath: string) {
  const key = getProjectPath(filepath) // 文件的项目路径，如 index.js
  if (depRelation.find(i => i.key === key)) {
    // 注意，重复依赖不一定是循环依赖
    return
  }
  // 获取文件内容，将内容放至 depRelation
  const code = readFileSync(filepath).toString()
  const { code: es5Code } = babel.transform(code, {
    presets: ['@babel/preset-env']
  })
  // 初始化 depRelation[key]
  const item = { key, deps: [], code: es5Code }
  depRelation.push(item)
  // 将代码转为 AST
  const ast = parse(code, { sourceType: 'module' })
  // 分析文件依赖，将内容放至 depRelation
  traverse(ast, {
    enter: path => {
      if (path.node.type === 'ImportDeclaration') {
        // path.node.source.value 往往是一个相对路径，如 ./a.js，需要先把它转为一个绝对路径
        const depAbsolutePath = resolve(dirname(filepath), path.node.source.value)
        // 然后转为项目路径
        const depProjectPath = getProjectPath(depAbsolutePath)
        // 把依赖写进 depRelation
        item.deps.push(depProjectPath)
        collectCodeAndDeps(depAbsolutePath)
      }
    }
  })
}
// 获取文件相对于根目录的相对路径
function getProjectPath(path: string) {
  return relative(projectRoot, path).replace(/\\/g, '/')
}
```

# Webpack Loader 原理

## 加载 CSS

思路：
1. 我们的打包器只能加载 JS
2. 我们想要加载 CSS
3. 如果能把 CSS 变成 JS，就能加载 CSS 了

在获取文件内容的时候，可以通过文件路径来判断是否是 css 文件，并稍作处理加载进 JS 中：

```typescript
let code = readFileSync(filePath).toString()
if (/\.css$/.test(filePath)) {
  code = `
    const str = ${JSON.stringify(code)}
    if (document) {
      const style = document.createElment('style')
      style.innerHTML = str
      document.head.appendChild(style)
    }
    export default str
  `
}
```

## CSS Loader

可以将上述代码变为 loader 的形式

```typescript
// css-loader.ts
const transform = (code) => `
    const str = ${JSON.stringify(code)}
    if (document) {
      const style = document.createElment('style')
      style.innerHTML = str
      document.head.appendChild(style)
    }
    export default str
  `
export default transform
```

```typescript
// bundler.ts
let code = readFileSync(filePath).toString()
if (/\.css$/.test(filePath)) {
  code = require('css-loader')(code)
}
```

## 尝试对 CSS Loader 进行优化

{% cq %}
单一职责原则：每个 Loader 只做一件事情
{% endcq %}

我们的 Loader 做了两件事情，第一是将 CSS 变为 JS 字符串，第二是将 JS 字符串放到了 style 标签里面，现在要对其进行拆分。

```typescript
// css-loader.ts
const transform = (code) => `
    const str = ${JSON.stringify(code)}
    export default str
  `
export default transform
```
```typescript
// style-loader.ts
const transform = (code) => `
    if (document) {
      const style = document.createElment('style')
      style.innerHTML = ${JSON.stringify(code)}
      document.head.appendChild(style)
    }
  `
export default transform
```
```typescript
// bunlder.ts
let code = readFileSync(filePath).toString()
if (/\.css$/.test(filePath)) {
  code = require('css-loader')(code)
  code = require('style-loader')(code)
}
```

上述代码是有问题的。最终得到的文件将不是我们想要的文件。

Loader 有不同的类型，像 sass-loader、less-loader 是将代码从一种语言转译为另外一种，这样的 loader 可以直接连接起来。但 style-loader 是插入代码而不是转译，所以需要寻找恰当的插入时机和位置——比如 css-loader 拿到结果之后。