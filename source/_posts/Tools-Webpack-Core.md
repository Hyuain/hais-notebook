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
const ast = parse(code, { sourceType: 'module' })
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

- 兼容策略 1：把代码全部放在 `<script type=module>` 中，这样会导致 IE 不兼容，并且会导致文件请求过多，因为每个被依赖的文件都需要被单独发出请求
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