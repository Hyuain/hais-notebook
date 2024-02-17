---
title: JavaScript Modules
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 的模块规范主要有 CommonJS (CJS)、Asynchronous Module Definition (AMD)、Universal Module Definition (UMD) 和 ES Modules (ESM) 。

> Refs:
>
> 1. [浏览器加载 CommonJS 模块的原理与实现](https://www.ruanyifeng.com/blog/2015/05/commonjs-in-browser.html)
> 2. [Modules in JavaScript – CommonJS and ESmodules Explained](https://www.freecodecamp.org/news/modules-in-javascript/#:~:text=CommonJS%20is%20a%20set%20of,support%20the%20use%20of%20CommonJS.)

<!-- more -->

# CommonJS

CommonJS 是一套实现 JavaScript 模组的标准，最早在 2009 年被提出。

- 用于 NodeJS，浏览器不支持 CommonJS
- NodeJS 过去只支持 CommonJS，现在也支持 ESModules 了

首先需要使用 `npm init -y` 创建一个 `npm` 项目，然后创建 `main.js` 文件：

```js
const testFunction = () => {
    console.log('Im the main function')
}

testFunction()
```

然后新建一个 `mod1.js` 文件：

```js
const mod1Function = () => console.log('Mod1 is alive!')
// module.exports 用来声明想要导出的东西
module.exports = mod1Function
```

然后便可以在 `main.js` 中这样使用：

```js
// require 用来声明引入的东西
mod1Function = require('./mod1.js')

const testFunction = () => {
    console.log('Im the main function')
    mod1Function()
}

testFunction()
```

也可以这样导出两个函数：

```js
const mod1Function = () => console.log('Mod1 is alive!')
const mod1Function2 = () => console.log('Mod1 is rolling, baby!')

module.exports = { mod1Function, mod1Function2 }
```

然后这样引入：

```js
({ mod1Function, mod1Function2 } = require('./mod1.js'))

const testFunction = () => {
    console.log('Im the main function')
    mod1Function()
    mod1Function2()
}

testFunction()
```

## CommonJS in Browser

浏览器不兼容 CommonJS 的根本原因在于缺少四个 NodeJS 环境的变量：`module` `exports` `require` `gloabl`。

我们可以通过提供这四个变量，让浏览器加载 CommonJS 模块，详见 [[Webpack]]

# AMD

AMD（Asynchronous Module Definition）最初是被设计来给前端用的，采用异步的方式加载模块，模块的加载不影响后面语句的进行，所有依赖模块的语句都定义在一个回调函数中，但他的定义更加繁琐一些：

```javascript
define(['dep1', 'dep2'], function (dep1, dep2) {
  // 通过返回函数来定义模组
  return function () {}
})

// or

define(function (require) {
  var dep1 = require('dep1'),
      dep2 = require('dep1')
  return function () {}
})
```

AMD 也采用 `require()` 加载模块，但不同于 CommonJS，还需要传给他一个回调函数：

```js
require([module], callback)
```

# UMD

UMD（Universal Module Definition）是一个通用模块定义，使得这些模块可以在客户端和服务端都能工作，UMD 基于 AMD 并能兼容 CommonJS。

一个典型的 UMD 模块需要这样定义：

```javascript
(function(root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD环境
        define(['dependency1', 'dependency2'], factory);
    } else if (typeof exports === 'object') {
        // CommonJS环境
        module.exports = factory(require('dependency1'), require('dependency2'));
    } else {
        // 全局环境（浏览器）
        root.moduleName = factory(root.dependency1, root.dependency2);
    }
}(this, function(dependency1, dependency2) {
    // 定义模块的内容
    var moduleName = function() {
        // ...
    };
    
    // 返回模块的导出对象
    return moduleName;
}));
```

# ESModules

ESModules 是 ES2015 中引入的一个新标准，旨在标准化 JavaScript 模块在浏览器中的实现。

我们需要在项目 `pakage.json` 文件中增加 `"type": "module"` 字段来在其中使用 ESModules，否则会得到一个错误提示：

```text
SyntaxError: Cannot use import statement outside a module
```

然后我们便可以在 `main.js` 中使用 `import` 语句来引入：

```js
import { mod1Function } from './mod1.js'

const testFunction = () => {
    console.log('Im the main function')
    mod1Function()
}

testFunction()
```

在 `mod1.js` 中使用 `export` 语句来导出：

```js
const mod1Function = () => console.log('Mod1 is alive!')
export { mod1Function }
```

此外还有一些小技巧，比如具名引入：

```js
import { mod1Function as funct1, mod1Function2 as funct2 } from './mod1.js'
import * as mod1 from './mod1.js' 
```

默认导出：

```js
export default mod1Function
```

如果我们想要在浏览器中使用的话，需要在 `script` 标签上增加 `type="module"`，否则也会得到错误提示。

可以通过 `import()` 表达式进行动态导入：

```js
import(modulePath)
  .then(obj => <module object>)
  .catch(err => <loading error, e.g. if no such module>)
```

