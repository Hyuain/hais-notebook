---
title: JavaScript
date: 2020-01-29 17:22:52
tags:
  - 入门
categories:
  - [前端, JavaScript]
---

- JavaScript 有 7 种数据类型，3 种变量声明的方式
- JavaScript 的数组不是典型的数组，而是一个对象；**元素的数据类型可以不同，内存不一定连续，是通过字符串下标（而不是数字下标）获取元素**。

<!-- more -->

# Startup

## History

1994 年，网景公司（当时叫 Mosaic Communications）发布了一款名为 **Mosaic Netscape** 的网页浏览器，在四个月内，这款浏览器就占据了四分之三的浏览器市场，并成为 1990 年代互联网的主要浏览器。

> 因为世界最早流行的图形接口网页浏览器 **NCSA Mosaic** 是美国国家超级电脑应用中心（NCSA）与 1993 年发布的，网景公司为了避免版权纠纷，将浏览器改名为 **Netscape Navigator**，而公司则改名为 **Netscape Communications**。

这款浏览器发布之后，网景意识到，**光有静态的页面是不行的，需一种网页脚本语言，使得浏览器可以与网页互动。**

1995 年，昇阳（Sun）正式向市场推出 Java，网景公司看到 Java 的前景，决定与之结盟，并在浏览器中支持 Java，但如果直接将Java作为脚本语言嵌入网页，只是因为这样会使HTML网页过于复杂。

同年，网景招募了布兰登（Brendan Eich），授意其开发一款 **“未来的脚本语言”** ，这种语言需要：“看上去与Java足够相似，但是比Java简单，使得非专业的网页作者也能很快上手。”——这个决定就排除了 Perl、Python、Tcl 或 Scheme 这些选项，同时也促成了 JavaScript 的诞生。

由于对 Java 不感兴趣，布兰登只用了十天时间就设计出了这款语言的原型，并命名为 **Mocha**，后续又改名为 **LiveScript**，但在 1995 年 12 月，公司为了蹭 Java 的热度，改名为 **JavaScript**。而事实上，JavaScript 和 Java 关系并不大。

> 总的来说，布兰登的设计思路是这样的：
>
> 1. 借鉴 C 的基本语法；
> 2. 借鉴 Java 的数据类型和内存管理；
> 3. 借鉴 Scheme，将函数提升到“第一等公民”（first class）的地位；
> 4. 借鉴 Self，使用基于原型（prototype）的继承机制。

由于 JavaScript 在浏览器上的大获成功，微软（Microsoft）在后续推出的 IE 3 上也使用了 **JScript** ——这与 JavaScript 是类似、但不同标准的语言。于是当年市场上出现了两者对峙的情况，网页设计者通常会在主页放上“用 Netscape可达到最佳效果 ”或“用 IE 可达到最佳效果”的标志。

1996 年 11 月，网景正式向 **欧洲计算机制造商协会（ECMA）** 提交语言标准；1997 年 6 月，ECMA 以 JavaScript 语言为基础制定了 ECMAScript 标准规范 ECMA-262。自然 JavaScript 也成为了 ECMAScript 最著名的实现之一。

由于只有短短十天的设计时间，而且世界上之前没有出现过结合了函数式编程和对象编程的语言，以及发展的迅速导致没有时间调整设计，JavaScript 成功成为了有着众多设计缺陷的语言，在这里不做细谈。

2001 年，微软发布 Windows XP，并捆绑了 IE 6。由于 Windows XP 迅速爆火以及长期的垄断，IE 6 也随之占据非常高的市场份额。前文已经说过，IE 6 对 JavaScript 支持并不好，同时 IE 6 对 CSS 标准的支持也不尽完善，导致前端技术的发展进入了漫长的蛰伏期。

2004 年，谷歌（Google）发布爆款应用 Gmail。这款应用在刚推出时，容量就比起其他受欢迎的电子邮箱服务如雅虎和微软的 Hotmail 多出过百倍，成为市场爆品，同时也让众多开发者看到了页面交互的巨大前景和可能性。

2005 年，Jesse 将谷歌用到的技术命名为 AJAX。

2006 年，至今为止最为长寿的 JavaScript 库—— jQuery，发布。

2008 年，谷歌发布 Chrome 浏览器；同年，Chrome 的使用率上升至 1%。其使用高性能 JavaScript 引擎 V8。

2009 年，Ryan 基于 V8 写了 Node.js。

2010 年，Isaac 基于 Node.js 写了 npm。

2010 年，TJ 受 Sinatra 启发，写了 Express.js。赶上了这几波顺风车的 JavaScript 迅速发展，并将触手伸向了后端。自此，JavaScript 也能胜任后端的一些工作了。

2012 年，Chrome 全球占有率达到 33%，超越 IE 跃居首位。

2015 年 12 月，Chrome 中国占有率达到 37%，超越 IE。

# Basic Syntax

## 表达式

- 表达式一般都有值，语句可能有也可能没有
- 语句一般会改变环境（声明、赋值），逗号表示语句没完

```js
// 表达式
1 + 2          // 值为 3
add(1, 2)      // 值为函数的返回值
console.log    // 值为函数本身
console.log(3) // 值为函数的返回值：undefined

// 语句
var a = 1
```

{% note warning %}
`retrun` 后面不能接回车，否则相当于返回 `undefined`
{% endnote %}

### 标识符

在 JavaScript 中，标识符只能包含字母或数字或下划线（“_”）或美元符号（“$”），且不能以数字开头。（有时候也可以用其他的 Unicode 字符，比如中文，比如 Emoji）

### 代码块 block

简单来说就是把代码用大括号包在一起

### for

```js
for(var i = 0; i < 5; i++){
  setTimeout(() => console.log(i))
}
// 5 5 5 5 5
for(let i = 0; i < 5; i++){
  setTimeout(() => console.log(i))
}
// 0 1 2 3 4
```

### label

```js
{ a:1 } // a 是一个 label，值是 1
```

## 值

### 字符存储

- 如何存数字？十进制转二进制，用十六进制（HEX）表示二进制
- 如何存字符？转换为数字，48 号表示 0，65 号表示 A，97 号表示 a
- 如何表示汉字（GB、GBK）？0000~FFFF，16 位，2 字节
- Unicode 已收录 13 万字符（大于 16 位），全世界通用，以后还会继续扩充；缺点：两个字节不够用了，至少要三个字节
- UTF-8：通过变长的存法，减小容量

### 7 种数据类型

> 4 基 2 空 1 对象

#### Number

> 64 位浮点数组成

##### 特殊值

`0` `+0` `-0`

`Infinity` `+Infinity` (1/0) `-Infinity` (1/-0)

`NaN`（0/0，但他还是一个数字，NaN不等于NaN）

##### 范围和精度

![JSNumber的存储](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Number-Range.png)

范围：`Number.MAX_VALUE` 到 `Number.MIN_VALUE`

精度：大概是 15 个十进制有效数字

#### String

> 每个字符两个字节（阉割版 UTF-8，两个字符定长）

##### 写法

单引号、双引号、反引号，引号不是字符串的一部分

##### 转义

`\r` 表示回车， `\uFFFF` 表示对应的 Unicode 字符， `\xFF` 表示前256个 Unicode 字符

##### 字符串的属性

字符串本来不应该有属性，只有对象才有属性，但是这个有渊源

- 长度： `s.length`
- 下标： `s[0]`
- base64 转码： `window.btoa` 编码， `window.atob` 反编码

#### Boolean

> 否定运算、相等运算、比较运算可以得到 bool 值

{% note warning %}

**5 个 falsy 值**

`undefined` `null` `0` `NaN` `''`

{% endnote %}

#### Symbol

> 每个 Symbol 都不一样

```js
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

#### undefined

如果声明了没有赋值，默认是 `undefined`

如果函数没有写 `return`，默认是 `undefined`

#### Null

习惯上把非对象空值写成 undefined，对象空值写成 null；

null 通常表示即该处不应该有值，undefined 通常表示"缺少值"，就是此处应该有一个值，但是还没有定义。

#### Object

{% note warning %}

数组、函数、日期都是 Object，不是单独的数据类型，但是使用 `typeof` 的时候，可以分辨出 function

{% endnote %}

### 数据类型检测

#### `typeof` `instanceof` `toString` 的比较

<table>
  <tr>
    <td colspan="2">类型</td>
    <td>举例</td>
    <td colspan="2">typeof</td>
    <td colspan="2">instanceof</td>
    <td colspan="2">Object.prototype.toString.call()</td>
  </tr>
  <tr>
    <td rowspan="4">基本类型</td>
    <td>Number</td>
    <td>1</td>
    <td>number</td>
    <td>√</td>
    <td>false*</td>
    <td>○</td>
    <td>[object Number]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>String</td>
    <td>hello'</td>
    <td>string</td>
    <td>√</td>
    <td>false*</td>
    <td>○</td>
    <td>[object String]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Boolean</td>
    <td>true</td>
    <td>boolean</td>
    <td>√</td>
    <td>false*</td>
    <td>○</td>
    <td>[object Boolean]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Symbol / new Symbol()</td>
    <td>new Symbol()</td>
    <td>symbol</td>
    <td>√</td>
    <td>false*</td>
    <td>○</td>
    <td>[object Symbol]</td>
    <td>√</td>
  </tr>
  <tr>
    <td rowspan="3">new 基本类型</td>
    <td>new Number()</td>
    <td>new Number(1)</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object Number]**</td>
    <td>√</td>
  </tr>
  <tr>
    <td>new String()</td>
    <td>new String('hello')</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object String]**</td>
    <td>√</td>
  </tr>
  <tr>
    <td>new Boolean()</td>
    <td>new Boolean('false')</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object Boolean]**</td>
    <td>√</td>
  </tr>
  <tr>
    <td rowspan="2">空值</td>
    <td>null</td>
    <td>null</td>
    <td>object</td>
    <td>×</td>
    <td>false*</td>
    <td>○</td>
    <td>[object Null]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>undefined</td>
    <td>undefined</td>
    <td>undefined</td>
    <td>√</td>
    <td>false*</td>
    <td>○</td>
    <td>[object Undefined]</td>
    <td>√</td>
  </tr>
  <tr>
    <td rowspan="6">对象类型</td>
    <td>普通对象</td>
    <td>{a: '1', b: '2'}</td>
    <td>object</td>
    <td>√</td>
    <td>true</td>
    <td>√</td>
    <td>[object Object]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Array</td>
    <td>[1, 2, 3]</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object Array]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Function</td>
    <td>function() {}</td>
    <td>function</td>
    <td>√</td>
    <td>true</td>
    <td>√</td>
    <td>[object Function]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Error</td>
    <td>new Error()</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object Error]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>Date</td>
    <td>new Date()</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object Date]</td>
    <td>√</td>
  </tr>
  <tr>
    <td>RegExp</td>
    <td>new RegExp()</td>
    <td>object</td>
    <td>○</td>
    <td>true</td>
    <td>√</td>
    <td>[object RegExp]</td>
    <td>√</td>
  </tr>
</table>
*尽管我们直接使用 `1 instanceof Number` 会出现错误，但是我们可以自定义 `instanceof` 方法，让他可以判断基本类型：

```js
class PrimitiveNumber {
  static [Symbol.hasInstance](instance) {
    return typeof instance === 'number'
  }
}
1 instanceof PrimitiveNumber // true 
```

**基本类型与用 `new Constructor` 构造的用对象包裹的基本类型实际上是不一样的，特别是在 `Boolean` 上，这点需要注意：

```js
const boolean = false
!!boolean // false

const newBoolean = new Boolean(false)
!!newBoolean // true
```

#### 自己实现一个 `instanceof`

```js
function myInstanceof(left, right) {
  if (typeof left !== 'object' || left === null) return false
  let proto = Object.getPrototypeOf(left) // 相当于 left.__proto__
  while(true) {
    if(proto === null) return false
    if(proto === right.prototype) return true
    proto = Object.getPrototypeOf(proto)
  }
}
```

#### `Object.is` 和 `===`

`Object.is` 修复了 `===` 的一些失误：

```js
function is(x, y) {
  if (x === y) {
    // 修复 +0 和 -0 相等的问题
    return x !== 0 || y !== 0 || 1 / x === 1 / y
  } else {
    // 修复 NaN 和 NaN 不相等的问题
    return x !== x && y !== y
  }
}
```

### 3 种变量声明的方式

#### var

- 过时的，不好的
- 函数作用域
- 可以重复声明
- 可以先使用再声明
- 全局声明的 `var` 变量会变成 `window` 的属性

#### let

- 新的，更合理的
- 遵循块作用域
- 不能重复声明
- 可以赋值，也可以不赋值
- 必须先声明再使用
- 全局声明的 `let` 变量不会再变成 `window` 的属性
- `let` 配合 `for` 循环有奇效

#### const

跟 `let` 几乎一样，但声明时必须赋值，且不能再更改

{% note warning %}
变量声明指定值的时候同时也指定了类型，但是 **值和类型都可以随意变化**
{% endnote %}

## 运算符

### 算术运算符

#### Number

- 余数 `-x % 7` 为 `-(x % 7)`
- 指数 `x ** 3`
- 自增 `a++` 表达式的值是 `a` 加之前的值， `++a` 表达式的值是 `a` 加之后的值
- 求值运算符 `+`，附属运算符 `-`

#### String

- 连接运算 `+`

{% note warning %}

`number + string`，变成字符串

`string - number`，变成数字

{% endnote %}

### 比较运算符

#### ==

- 模糊相等，发生自动类型转换，别用两个等于
- JavaScript 三位一体

#### ===

全等，基本类型看值是否相等，对象看地址是否相等

- `[] !== []`
- `{} !== {}`
- `NaN !== NaN`

### 布尔运算符

```js
console && console.log && console.log('hi') // 防止 console.log 报错（防御性编程）
a = a || 100 // 但是五个 falsy 值都会让 a 为假，因此有 bug

// 可以用函数来赋初值
function add(n = 0) {
  return n + 1
}
```

#### 短路逻辑

##### &&

如果前面是 **真的**，就执行后面的（若前面是假的，表达式的值为前面；若前面是真的，表达式的值为后面）

```js
window.f1 && console.log('f1 存在')
console && console.log && console.log('hi') // 因为 IE 没有 console.log，所以可以这样写防止出错
```

##### ||

如果前面是 **假的**，就执行后面的

```js
a = a || 100 // 可以用于设置保底值
```

### 二进制运算符

#### 或与否

```js
(0b1111 | 0b1010).toString(2) // 1111
(0b1111 & 0b1010).toString(2) // 1010
(~0b1010).toString // 涉及到补码，留坑
```

#### 异或

```js
(0b1111 ^ 0b1010).toString(2) // 101
```

{% note warning %}

按位取反可以用 `^1`

{% endnote %}

#### 左移和右移

```js
(0b0011 >> 1).toString(2) // 1
(0b0010 << 1).toString(2) // 100
```

#### 头部补零的右移运算符

```js
(0b0011 >>> 1).toString(2) // 1
```

#### 使用运算符判断奇偶

```js
偶数 & 1 === 0
奇数 & 1 === 1
```

#### 使用 `~` `>>` `<<` `>>>` `|` 来取整

> 位运算不支持小数，会自动抹去

```js
~~ 6.83 // 6
6.83 >> 0
6.83 << 0
6.83 | 0
6.83 >>> 0
```

#### 使用 `^` 来交换 `a` `b` 的值

```js
a ^= b
b ^= a
a ^= b

// 新版语法
[a, b] = [b, a]
```

### 其他运算符

#### 点运算符

```js
对象.属性名 = 属性值
```

#### void 运算符

> 求表达式的值或执行语句，然后 `void` 的值总为 `undefined`

```js
void 表达式或语句
```

```html
<!-- 防止假动作 -->
<a href="example.com" onclick="console.log(hi); return false;">点击</a>

<!-- 也可以用 -->
<a href="javascript:void(console.log('hi'))">点击</a>

<!-- 或者 -->
<a href="javascript:;">点击</a>
```

#### 逗号运算符

> 表示取后面的值

```js
var a = (1, 2) // a 为 2

let f = x => { console.log('hi'); return x + 1 }
let f = x => (console.log('hi'), x + 1)
// 跟上面是一样的，先执行 console.log，再让 return 为x + 1
```

### 优先级

圆括号的优先级最高

## ES6

作用域、箭头函数、默认参数、展开运算符、模板字符串、解构赋值、import export、class、for of、Generator、新的数据类型（Symbol、Map、Set）

## Generator

### 生成器执行流程

生成器是一个带星号的“函数”（其实他并不是真正的函数），可以通过 yield 关键字暂停执行和恢复执行

```js
function* gen() {
  console.log('enter')
  let a = yield 1
  let b = yield (function() {
    return 2
  })
  return 3
}

var g = gen()
```

1. 调用 `gen()` 之后，程序会阻塞住，不会执行任何语句
2. 调用 `g.next()` 之后，程序会继续执行，直到遇到 `yield`
3. `next` 方法会返回一个对象，有 `value` 和 `done` 属性，`value` 为 当前 `yield` 后面的结果，`done` 表示是否执行完，遇到 `return` 后，`done` 变为 `true`

### yield*

```js
function* gen1() {
  yield 1
  yield* gen2()
  yield 4
}
function* gen2() {
  yield 2
  yield 3
}

// 会按 1 2 3 4 的顺序执行
```

### 生成器的实现机制：协程

一个线程可以存在多个协程，协程不受操作系统的管理，而是被具体的应用程序代码所控制，因此没有进程的上下文切换的开销，性能高。

一个线程一次只能执行一个协程，可以通过 `yield` 暂停当前协程，并将 JS 线程的控制权转移给另一个协程。

### Generator 与异步

#### thunk 函数

thunk 函数即偏函数，核心是接受一定的参数，生产出定制化的函数，然后使用定制化的函数去完成功能，相当于一群函数的抽象和封装

#### Generator 与异步

##### thunk

```js
const readFileThunk = (filename) => {
  return (callback) => {
    fs.readFile(filename, callback)
  }
}

const gen = function* () {
  const data1 = yield readFileThunk('001.txt')
  console.log(data1.toString())
  const data2 = yield readFileThunk('002.txt')
  console.log(data2.toString())
}

let g = gen()
// value 值即为 thunk 生成的定制化函数
g.next().value((err, data1) => {
  // 拿到上一次的结果，调用 next，将结果作为参数传入
  g.next(data1).value((err, data2) => {
    g.next(data2)
  })
})

// 可以将上面的代码进行封装，减少嵌套
function run(gen) {
  const next = (err, data) => {
    let res = gen.next(data)
    if (res.done) return
    res.value(next)
  }
  next()
}
run(g)
```

##### Promise

也可以用 Promise 来实现：

```js
const readFilePromise = (filename) => {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, (err, data) => {
      if (err) {
        reject(err)
      } else {
        resolve(data)
      }
    })
  }).then(res => res)
}

const gen = function* () {
  const data1 = yield readFilePromise('001.txt')
  console.log(data1.toString())
  const data2 = yield readFilePromise('002.txt')
  console.log(data2.toString())
}

let g = gen()
function getGenPromise(gem ,data) {
  return gen.next(data).value
}
getGenPromise(g).then(data1 => {
  return getGenPromise(g, data1)
}).then(data2 => {
  return getGenPromise(g, data2)
})
```

# Object

## 创建对象

对象有以下这样几种写法：

```js
// 对象的简便声明
let obj = {
  'name': 'harvey',
  'age': 18
}
// 对象的标准写法
let obj = new Object({
  'name': 'harvey',
  'age': 18
})
// 对象也可以作为参数直接传入函数
console.log({
  'name': 'harvey',
  'age': 18
})
```

在这里我们需要注意以下几个细节：

- 键名是字符串，不是标识符，可以是任意字符
- 引号可以省略，省略之后就只能写标识符或者以数字开头
- **就算引号省略了，键名也还是字符串**（不一定，键名也可能是函数、日期、正则等）
- 奇怪的属性名： `1e2` 会变成 `'100'`， `.234` 会变成 `'0.234'`， `0xFF` 会变成 `'255'`
- 变量也可以作为属性名，比如：`let obj = { [p1]: 'harvey' }`，这样就会用 `p1` 里面的值了，**中括号里面的东西都会先求值**

## 对象的属性

### 隐藏的属性

- **每一个** 对象都有一个隐藏属性 `__proto__`，这个属性存着 **一个对象的地址**，这个对象包含了这类对象（普通对象、数组、函数等）的 **共有属性**

- `__proto__` 里面存的实际上就是 **原型的地址**

- 因此，**每一个对象都有原型**

- 比如 `obj = {}`，他的原型的地址就存储在 `obj.__proto__` 中，而`obj.__proto__` 也是一个对象，因此他也有原型，但我们规定，他的原型值为 `null`

### 属性的删除

有以下两种删除对象属性的方法：

1. `obj.name = undefined`，这样做只会删除属性的值，不会把属性完全删除
2. `delete obj.name`，同时删除属性名和属性值，或者用 `delete obj['name']` 也是可以的

删除完成后可以对删除的结果进行检查：

1. `'name' in obj`，检查 `'name'` 是不是 `obj` 的属性名，如果是用上面的第一种方法删除，检查的结果将是 `true`；如果是第二种方法删除，则会返回 `false`；注意属性名有引号（因为属性名实际上是字符串）
2. `'name' in obj && obj.name === undefined`，检查是否含有属性名且值为 `undefined`，如果是上面第一种方法删除，则会返回 `true`

### 属性的查看

- `Object.keys(obj)`，查看 `obj` 的 **自身** 属性名
- `Object.values(obj)`，查看 `obj` 的  **自身** 属性值
- `Object.entries(obj)`，返回结果包含两个数组，第一个数组是 `obj` 的 **自身** 属性名，第二个数组是 `obj` 的 **自身** 属性值
- `console.dir(obj)`，查看 `obj` 的 **自身属性 + 共有属性**
- `in`，查看是不是 **自身属性 + 共有属性** （相当于所有属性）
- `obj.hasOwnProperty('toString')`，查看 `'toString'` 是不是 `'obj'` **自身的** 属性

{% note warning %}

`obj.name` 等价于 `obj['name']`，不等价于 `obj[name]`

{% endnote %}

### 添加或修改属性

直接赋值：

```javascript
obj.name = 'harvey'
obj['name'] = 'harvey'
obj['na' + 'me'] = 'harvey' // 因为属性名本质是字符串，上面三句话实际上是一样的
```

批量赋值：

```javascript
Object.assign(obj, { p1: 1, p2: 2, p3: 3 })
```

但是，不能直接修改共有属性（原型上的属性）：比如，不能通过 `obj.toString` 来修改原型上的 `'toString'`，这样只会为 `obj` 增添一个本身的 `'toString'` 属性，而不会修改原型，除非这样写代码（但是这是不推荐的）：

```js
// 修改原型的属性
obj.__proto__.toString = Object.prototype.toString =

// 修改原型
let person = Object.create(common)
// 相当于
person.__proto__ = common // 原型链增加一个环节
```

## 对象的原型

{% note warning %}

所有的函数自带 prototype（箭头函数没有 prototype）

prototype 中自带 constructor

constructor 里面的东西就是函数的内容

{% endnote %}

### new X()

1. 自动创建空对象
2. 自动为空对象关联原型，原型的地址为 `X.prototype`
3. 自动将空对象作为 `this` 关键字运行构造函数
4. 自动 `return this`（也就是说可以接着写 `new X().getName()`）

```js
// 自己实现一个类似 new 的函数
function X(a, b) {
  this.a = a + 1
  this.b = b + 1
}

const y = new X(1, 2)

function NEW(fun, ...args) {
  const newObj = Object.create(fun.prototype)
  // 相当于
  // let newObj = {}
  // newObj.__proto__ = fun.prototype
  const result = fun.apply(newObj, args)
  // 原版的 new 中，如果构造函数返回一个对象，则 new 也返回一个对象；如果构造函数返回一个简单类型，则 new 返回刚刚创建的新对象
  return typeof result === 'object' ? result : newObj
}

const z = NEW(X, 3, 4)
```

### 构造函数 `X`

- `X` 自身用于添加新对象的**自身的属性**
- `X.prototype` 负责保存对象的**共用属性**

一般来讲有这样的代码规范：

- 所有构造函数首字母大写，被构造出来的对象首字母小写
- `new` 后面的函数使用名词；其他的函数一般用动词开头

### 如何确定一个对象的原型？

```js
对象.__proto__ === 其构造函数.prototype
```

### 数据类型与对象的分类

> 数据类型是 JavaScript 数据的类型，一共有 7 种；
>
> 对象的分类则有无数种，常见的有 Array、Function、Date、RegExp 等

我们常见的有这几种类型的对象：

- 数组对象
  - 自身属性：`'0'` `'1'` `'2'` `'length'`
  - 共有属性：`'push'` `'pop'` `'shift'` `'unshift'` `'join'`

- 函数对象
  - 自身属性：`'name'` `'length'`
  - 共有属性：`'call'` `'apply'` `'bind'`

## 其他问题

> **Q: window 是谁构造出来的？**
>
> A: 可以通过 constructor 属性看出构造者

> **Q: window.Object 是谁构造的？**
>
> A: window.Function，所有的函数都是 window.Function 构造的

> **Q: window.Function 是谁构造的？**
>
> A: window.Function，所有的函数都是 window.Function 构造的，浏览器构造了 Function，然后指定它的构造者是自己

# Array

## 获得新数组

### 将类数组转化为数组

```js
args = Array.prototype.slice.call(arguments)
args = Array.from(arguments)
args = [...arguments]
args = Array.prototype.concat.apply([], arguments)
```

### 新建数组

```javascript
// 数组的简便定义
let arr = [1, 2, 3]

// 数组的标准写法
let arr = new Array(1, 2, 3)

// 或者只传入一个参数，表示数组的长度
let arr = new Array(3)
```

### 转化为数组

```javascript
// 以','分隔的字符串
let arr = '1,2,3'.split(',')

// 没有间隔的字符串
let arr = '123'.split('')
Array.from('123')

// 由对象转换
let arr = new Array(3)

// 数组转化为字符串
let str = [1, 2, 3].join(',')
```

数组对象除了 `__proto__` 之外，还包括 **索引** 和 **长度（`length`）** 这两个自身属性。

{% note warning %}

**伪数组**：伪数组的原型链中没有数组的原型

比如 `let divList = document.querySelector('div')` 将得到一个伪数组；一个普通的对象只是加上 `length` 属性，也将得到一个伪数组。

通常我们需要把它转化为数组来使用

`let divArray = Array.from(divList)`

{% endnote %}

### 合并数组

```js
let arr3 = arr1.concat(arr2) // 生成新数组，原数组不变
```

### 截取数组

```js
let arr2 = arr1.slice(2) // 生成新数组，原数组不变
```

### 复制数组

```js
let arr2 = arr1.slice(0) // 生成新数组，原数组不变
```

## 删除元素

```js
arr = ['a', 'b', 'c'];  delete arr['0']
// arr 为 [ empty, 'b', 'c']，如果3个都是 empty，称为稀疏数组，不推荐

// 改 length 也可以删除数组的元素，不推荐

arr.shift()
// 删除最开始的元素，并返回他，arr 被修改

arr.pop()
// 删除最后一个元素，并返回他，arr 被修改
 
arr.splice(2, 3)
// 从 2 开始，删除 3 个，并返回删除的部分，arr 被修改

arr.splice(2, 3, 'x', 'y')
// 从 2 开始，删除 3 个，增加 'x' 和 'y'
// 并返回删除的部分，arr 被修改
```

## 查找元素

### 遍历元素

```js
Object.keys(arr) // 不推荐
Object.values(arr) // 不推荐
for in // 不推荐

for(let i = 0; i < arr.length; i++) {
  console.log(`${i} : ${arr[i]}`)
}

arr.forEach(function(item, index, array) {
  console.log(`$(index) : $(item)`)
})

// 以上两种基本没有区别
// 但 for 关键字有 continue 和 break，forEach 没有
// for 是块级作用域，forEach 是函数作用域
```

### 查找单个元素

```js
arr.indexOf(item) // 有就会返回 index，没有就会返回 -1

arr.find(item => item % 2 === 0) 
// 会返回第一个符合条件的元素
arr.findIndex(item => item % 2 === 0)
// 会返回第一个符合条件的元素对应的索引
```

{% note warning %}

**索引越界**

```js
arr[arr.length] === undefined
a[-1] === undefined
```

{% endnote %}

## 增加元素

```js
arr.push()
// 在尾部添加，返回数组长度，arr 被修改

arr.unshift()
// 在头部添加，返回数组长度，arr 被修改

arr.splice(8, 0, 'x', 'y')
// 增加 'x' 和 'y'，并返回删除的部分（[]），arr 被修改
```

## 修改元素

```js
arr.reverse() // arr 被修改
arr.sort() // arr 被修改

arr.sort( function(a, b) {
  if ( a > b ) return 1
  else if ( a === b ) return 0
  else return -1
})
// 返回 1，a 在 b 之后
// 返回 0，不变
// 返回 -1，b 在 a 之后

arr.sort((a, b) => a.score - b.score) // 按 score 从小到大排序
```

## 数组变换

得到新数组，原数组不变

```js
// map： n 变 n
arr.map(item => item * item)

// filter：n 变少
arr.filter(item => item % 2 === 0)

// reduce：n 变 1
arr.reduce((sum, item) => sum + item, 0)
arr.reduce((result, item) => result.concat(item * item), [])
arr.reduce((result, item) =>
  result.concat(item % 2 === 1 ? [] : item), [])
```

# Function

## 定义一个函数

### 具名函数

```js
function 函数名(形参) { 函数体 }
```

### 匿名函数

```js
let a = function(形参) { 函数体 } // 右边的部分也叫函数表达式
let a = function fn(形参){ 函数体 }
// 如果函数的声明是在 = 右边
// fn 的作用域只能在 = 右边
// 别的地方不能用 fn 这个名字
```

### 使用构造函数

```js
let a = new Function()
```

### 箭头函数

```js
输入参数 => 输出参数
( 输入参数1, 输入参数2 )  => 输出参数
( 输入参数1, 输入参数2 )  => {
  语句1
  语句2
  return 语句
}
x => ({ name: '...' }) // 如果要返回对象，就要加圆括号
```

### 函数提升

函数可以先使用再声明，因为 function 会跑到最前面

但是 `let fn = function(){}` 不是函数提升，他不能先使用再声明，因为这句话是赋值，右边的匿名函数不会提升

## 作用域

- **全局变量与局部变量**：在 **顶级作用域** 声明的变量是全局变量（比如 window 上的变量是全局变量）；其他的都是局部变量
- **就近原则**：如果有多个作用域有同名变量 `a`，那么查找 `a` 的声明时，就向上取最近的作用域
- **作用域的确定与函数的执行无关（JavaScript 的作用域为静态作用域），但变量的值在函数执行的时候才能确定**

### 闭包

> 函数用到了外部的变量，则函数+这个变量=闭包，作用域遵循就近原则

闭包的作用：隐藏局部变量，暴露操作函数

```js
const createAdd = () => {
  let n = 0
  return () => {
    n += 1
    console.log(n)
  }
}

const add = createAdd()
add() // 1
add() // 2
```

闭包的缺点：容易内存泄露。注意，虽然闭包并不会造成内存泄露，真实原因是 JS 引擎的实现有问题。

## 参数

### 参数传递

复制内存图中 stack 里面的东西（简单类型复制值，对象复制地址）

### 形参

实际上就是变量声明： `var x = arguments[0]`

{% note warning %}

形参可多可少

{% endnote %}

```js
function add(x) {
	return x + arguments[1]
}
add(1, 2) // 3

function add(x, y, z) {
	return x + y
}
add(3, 4) // 7
```

## 返回值

- 每个函数都有返回值，默认是 `undefined`
- 只有函数才有返回值

## 调用栈

- JS 在调用函数之前，需要把函数的环境 push 到一个数组（调用栈）里面，等函数执行完之后把环境 pop 出来，然后 return 到之前的环境中
- 递归很容易把栈压满：爆栈
- 调用栈最长有多少？Chrome 12578；Firefox 26773；Node 12536

## this 和 arguments

> 关于 this 更多的内容可以看看这篇文章—— [再看 this](https://hais-teatime.com/post/2019-12-24-this/)。

### arguments

调用函数即传入 arguments， arguments 是包含传入参数的伪数组

### this

- `this` 的存在是为了解决函数获取一个对象的引用的问题
- 如果不给任何的条件， `this` 为 `window`
- 如果想要指定 `this`，要用 `fn.call(xxx, 1, 2, 3)` 传入 `this` 和 `arguments`（如果传入的 `this` 不是对象，将会默认封装成对象，除非加上 `'use strict'`，JS的糟粕）

```js
function fn() {
  'use strict'
  console.log('this: ' + this)
}
fn.call(1) // 'this: 1'
fn.call(undefined) // 'this: undefined'
```

#### 两种调用函数的方法

##### 隐式传递

```js
person.sayHi()
// 会自动地把 person 传到函数里，作为 this
```

##### 显式传递

```js
person.sayHi.call(person)
// 手动把 person 传到函数里，作为 this
// apply 跟 call 的区别就是后面要加中括号（参数传的是数组）
```

#### 绑定 this

```js
function f1(p1, p2){ console.log(this, p1, p2) }
let f3 = f1.bind( {name:'hai'}, 'hi' )
f3()  // 等价于 f1.call( {name:'hai'}, 'hi' )
      // 相当于让 f3 的 this 和 arguments 永远等于这个
```

#### 箭头函数

没有 `arguments` 和 `this`，里面的 `this` 就是外面的 `this`，就算加 `call` 也没有

```js
console.log(this) // window
let fn = () => console.log(this)
fn() // window
fn.call( {name:'hai'} ) // window
```

## 立即执行函数

只想要一个局部变量，而不想要一个函数

```js
+/-/1*/! function (){ var a = 2; console.log(a) } ()
// 声明了一个全局函数，立即调用
```

# JavaScript World

## 开机

### 固件

- 固件是固定在主板上的存储设备，里面有开机程序
- 开机程序会将文件里的 OS 加载到内存里运行

### 操作系统（以 Linux 为例）

- 首先加载操作系统内核
- 然后启动初始化进程，编号为 1，每个进程都有 PID
- 启动系统服务：文件、安全、联网
- 等待用户登录：输入密码登录 / ssh 登录
- 登录后，运行 shell，用户就可以和操作系统对话了
- bash 是一种 shell，图形化界面可认为是一种 shell

## 打开浏览器

### chrome.exe

- 运行 chrome.exe 文件
- 开启 chrome 进程，作为主进程
- 主进程会开启一些辅助进程，如网络服务、GPU 加速
- 每新建一个标签，就有可能开启一个子进程

### 浏览器的功能

- 发起请求、下载 HTML、解析 HTML、下载 CSS、解析 CSS、渲染界面、下载 JS、解析 JS、执行 JS
- 功能模块：用户界面、渲染引擎、JS 引擎、存储等，这些功能模块属于不同的线程
  - JS 通过**跨线程通信**，使渲染引擎重新渲染
  - JS 是单线程的
  - DOM 操作慢：跨线程通信慢

### JavaScript 引擎

- 浏览器用的什么 JS 引擎？
  - Chrome 用的是 V8 引擎，C++ 编写
  - 网景用的是 SpiderMonkey，后被 Firefox 使用，C++
  - Safari 用的是 JavaScriptCore
  - IE 用的是 Chakra（JScript 9）
  - Edge 用的是 Chakra (JavaScript)
  - Node.js 用的是 V8 引擎
- 主要功能
  - 编译：把 JS 代码翻译为机器能执行的字节码或机器码
  - 优化：改写代码，使其更加高效
  - 执行：执行上面的字节码或机器码
  - 垃圾回收：把 JS 用完的内存回收，方便之后再次使用

### 运行 JavaScript 代码

- 准备工作
  - 提供 API： window、document、setTimeout（这些功能成为运行环境 runtime env）
- 内存图
  - ![内存图](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/RAM-Graph.png)
  - 红色区域（存放数据，但不会存变量名）
    - Stack 栈：每个数据顺序存放，非对象都存在 Stack
    - Heap 堆：每个数据随机存放，对象都存在 Heap（数组、函数）

### 堆内存与栈内存

JS 的内存空间分为

- 栈（Stack）：存放简单类型，LIFO
- 堆（Heap）：存放引用类型，是一种经过排序的树形结构，每个节点都有一个值，通常说的堆是指二叉堆，存取随意
- 池（一般也被归类为栈中）：存放常量，故又称常量池

#### 堆与栈的优缺点

- 栈内存系统效率高，因为堆内存需要分配空间和地址，还要把地址存在栈内存中，因此效率低
- 引用类型变量大小不确定，如果放到栈内存中就无法轻易改变自己的大小，同时也浪费空间

闭包中的变量保存在堆内存中，而不是栈内存中，因此函数调用之后闭包仍然能够引用到函数内的变量

#### 闭包与堆内存

```js
function A() {
  let a = 1
  function B() {
    console.log(a)
  }
  return B
}
let res = A()
```

函数 A 弹出调用栈之后，函数 A 中的变量这个时候是存储在堆上的，现在的 JS 引擎可以通过逃逸分析辨别出哪些变量需要存储在堆上，哪些需要存储在栈上

### V8 执行一段代码的过程

1. 首先通过词法分析和语法分析生成 **AST**
2. **解释器** 将 AST 转换为字节码
3. 由 **解释器** 逐行执行字节码，遇到热点代码启动 **编译器** 进行编译，生成对应的机器码，优化执行效率

#### 生成 AST

##### 词法分析

主要是把一行行代码分解成一个个 token：

```js
let name = 'harvey'
```

会把句子拆分成四个部分（四个 token）：关键字 let、变量名 name、赋值 =、字符串 'harvey'

##### 语法分析

将生成的这些 token 数据，根据一定的语法规则转化为 AST，之后会生成执行上下文。
babel 实际上就是将 ES6 代码解析生成  ES6 的 AST，然后再转换为 ES5 的 AST，然后再转换为 ES5 的代码。

##### 生成字节码

生成 AST 之后，直接通过 V8 解释器（Ignition） 生成字节码，但是字节码并不能让机器直接运行——之所以不直接转化为机器码，是因为机器码空间占用太大了。

字节码比机器码轻量很多，是介于 AST 和机器码之间的一种代码，但是与特定类型的机器码无关，字节码需要通过解释器将其转换为机器码然后执行。但和原来不同的是，现在不用一次性将全部的字节码都转换成机器码，而是通过解释器来逐行执行字节码，省去了生成二进制文件的操作，这样就大大降低了内存的压力。

#### 执行代码

热点代码：在执行字节码的过程中，如果有一段代码反复出现，则记为 **热点代码**（HotSpot），这段代码会被编译成机器码保存起来。在这样的机制下，代码执行的时间越久，那么执行效率会越来越高，因为有越来越多的字节码被标记为热点代码。

JS 实际上并不是一门完全的解释型语言，因为字节码不仅配合了 **解释器**，还要配合 **编译器**。编译器和解释器的根本区别在于前者会编译生成二进制文件而后者不会。

## JavaScript World

{% note warning %}

**JS 公式：`对象.__proto__ === 其构造函数.prototype`**

**根公理：`Object.prototype` 是所有对象的（直接或间接）原型**

**函数公理：所有函数都是由 `Function` 构造的**

{% endnote %}

![JSWorld-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-1.png)
![JSWorld-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-2.png)
![JSWorld-3](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-3.png)
![JSWorld-4](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-4.png)

# DOM

## 获取节点

```js
window.id // 或者直接 id

document.getElementById('idxxx')
document.getElementsByTagName('div')[0]
document.getElementsByClassName('div')[0]

document.querySelector('div>span:nth-child(2)')
document.querySelectorAll('.red')
```

### 获取特定的元素

- 获取 html： `document.documentElement`
- 获取 head： `document.head`
- 获取 body： `document.body`
- 获取 window： `window`
- 获取所有元素： `document.all`，第 6 个 `falsy` 值，别的浏览器为了不使用为了 IE 设计的代码（这个是 IE 发明的）

### 获取的元素的原型

div 的原型链：

`HTMLDivElement.prototype` → `HTMLElement.prototype` → `Elment.prototype` → `Node.prototype` → `EventTarget.prototype` → `Object.prototype`

### 节点（Node）和元素（Element）的区别

使用 x.nodeType 可以得到一个数字：

- 1 表示 Element（也叫 Tag ）
- 3 表示 Text
- 8 表示 Comment
- 9 表示 Document
- 11 表示 Document Fragment

### 获取附近的元素

- 查爸爸： `div.parentNode` 或者 `div.parentElement`
- 查子代： `div.childNodes` 或者 `div.children`，推荐使用后者，因为前者包括了文本节点，比如空格，两者都会实时变化
- 查兄弟姐妹： `div.parentNode.childNodes` 或者 `div.parentNode.children`，然后再排除自己
- 查看老大： `div.firstChild`
- 查看老幺： `div.lastChild`
- 查看上一个哥哥： `div.previousSibling`（有可能查到文本节点，可以用 `div.previousElementSibling`）
- 查看下一个弟弟： `div.nextSibling`（有可能查到文本节点，可以用 `div.nextElementSibling`）

```js
// 遍历一个 div 里面所有的元素
travel = (node, fn) => {
  fn(node)
    if(node.children) {
      for(let i = 0; i < node.children.length; i++) {
      travel(node.children[i], fn)
    }
  }
}
travel(div1, (node) => console.log(node))
```

## 创建节点

### 创建一个标签

```js
let div1 = document.createElement('div')
document.createElement('style')
document.createElement('script')
document.createElement('li')
```

### 创建一个文本节点

```js
text = document.createTextNode('你好')
```

### 在标签里面插入文本

```js
div1.appendChild(text1)
div1.innerText = '你好'
// 或者
div1.textContent = '你好'
//但是不能用
div.appendChild('你好')
```

### 插入到页面中

- `document.body.appendChild(div)` 或者 `已经在页面中的元素.appendChild(div)`

```js
// 页面中有 div#test1 和 div#test2
let div = document.createElement('div')
test1.appendChild(div)
test2.appendChild(div)
// div 最终只会出现在 test2 里面
```

- 让两个地方都有： `let div2 = div1.cloneNode(true)`，如果后面为 `true` 则使用深拷贝，所有的后代也会被拷贝

## 删除节点

```js
div.parentNode.removeChild(div)
div.remove()
```

{% note warning %}

如果一个 Node 被移除了页面（DOM）树，它还可以再被放回去，它暂存在了内存里面，如果想删除，`div = null`，一会儿会自动回收

{% endnote %}

## 编辑节点

### 编辑属性

```js
div.className = 'red blue'// 会覆盖掉之前的 class，要用 div.className += ' red'

div.classList.add('red')

div.style = 'color: blue'
// 会覆盖掉之前的 style，一般就写 style 的一部分
// 比如 div.style.color = 'blue'，注意大小写问题：div.style.backgroundColor

div.setAttribute('data-x', 'test')

div.getAtrribute('data-x') // 强加的，这样写也行
```

{% note warning %}

如果用 `a.href` 获取属性，然后是相对路径的话，他会把域名一起弄下来

如果用 `a.href.getAttribute('href')` 只获取属性里面写了的部分

{% endnote %}

### 编辑事件

- `div.onclick` 默认为 `null`，如果改成 `fn`，那么在点击的时候就会调用 `fn.call(div, event)`，`event` 包含了点击事件的所有信息，比如坐标
- `div.addEventListener`

### 编辑内容

#### 编辑文本内容

```js
test.innerText = 'hi'
test.textContent = 'hi'
```

#### 修改 HTML 内容

```js
test.innerHTML = '' // 里面的字符不能超过两万个
```

#### 修改标签

```js
test.innerHTML = '' // 先清空
test.appendChild(div) //再加内容
```

### 跨线程

一个栗子：

```css
.start{
  border: 1px solid red;
  width: 100px;
  height: 100px;
  transition: width 1s;
}
.end{
  width: 200px;
}
```

```js
test.classList.add('start')
test.clientWidth // 如果没有这句话，上下两句话就会合并，这句话会触发重新渲染
test.classList.add('end')
```

#### 属性同步

- 标准属性，会自动同步，比如 `id` `className` `title`
- `data-*` 属性，也会自动同步
- 非标准属性，不会同步到页面里，只会停留在JS 线程中，而不会自动同步到页面上

![JSProperty-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Property-1.png)
![JSProperty-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Property-2.png)

{% note warning %}

自定义属性最好以 `data-` 为前缀

{% endnote %}

#### Property 和 Attribute

- Property 是 JS 线程中 div1 的所有属性
- Attribute 是 渲染引擎中 div1 对应标签的属性
- 大部分时候，同名的 Property 和 Attribute 值相等
- 如果不是标准属性，那么他们俩只会在一开始的时候相等
- 但注意 Attribute 只支持字符串，而 Property 支持字符串、布尔等类型

# Asynchronous

## AJAX

> AJAX 即 Asynchronous JavaScript and XML（异步的 JavaScript 与 XML 技术），其实就是一套综合了多项技术的浏览器端网页开发技术。Google 在它多个著名的交互应用程序中使用了这套技术，如 Google 讨论组、Google 地图、Google 搜索建议、Gmail 等，这使得人们看到了前端领域新的可能性。
>
> 传统的 Web 应用允许用户端填写表单（form），当提交表单时就向网页服务器发送一个请求；服务器接收并处理传来的表单，然后送回一个**新的网页**。
>
> 但这个做法浪费了许多带宽，因为在前后两个页面中的大部分 HTML 码往往是相同的。由于每次应用的沟通都需要向服务器发送请求，应用的回应时间依赖于服务器的回应时间。这导致了用户界面的回应比本机应用慢得多。
>
> 与此不同，AJAX 应用可以仅向服务器发送并取回必须的数据，并在客户端采用 JavaScript 处理来自服务器的回应。因为在服务器和浏览器之间交换的数据大量减少，服务器回应更快了。同时，很多的处理工作可以在发出请求的客户端机器上完成，因此Web服务器的负荷也减少了。

简单来讲：

1. AJAX 是浏览器上的功能，浏览器可以使用 AJAX 发请求、收响应；
2. 浏览器在 `window` 上面加了一个 `XMLHttpRequest`，方便开发者通过 JS 来发请求、收响应，这是一个构造函数

### 加载 CSS / JS / HTML / XML

如果想要使用 `XMLHttpRequest` 让浏览器来帮我们加载 CSS / JS / HTML / XML，一共需要进行四步：

1. 创建 `XMLHttpResquest` 对象
2. 调用他的 `open()` 方法
3. 使用 `onreadystatechange` 监听他的成功和失败事件
4. 调用他的 `send()` 方法

```js
const request = new XMLHttpRequest()
request.open('GET', '/{url}')
request.onreadystatechange = () => {
  if (request.readyState == 4) {
    if (request.status >= 200 && request.status < 300) {
      alert('success')
    } else {
      alert('error')
    }
  }     
}
request.send()
```

HTTP 里面可以装 HTML / CSS / JS / XML 等，但得知道怎么解析他们：

- 拿到 CSS 之后生成 `<style>` 标签
- 拿到 JS 之后生成 `<script>` 标签
- 拿到 HTML 之后使用 `innerHTML` 和 DOM API，可以新建一个 `div`
- 拿到 XML 之后的 `request.responseXML` 实际上已经是一个 DOM 节点了

### JSON

> JavaScript Object Natation，JSON 不是对象，而是一种标记语言（就像 HTML、XML、Markdown），用来展示数据，[JSON 中文官网](http://json.org/json-zh.html)

#### JSON 的数据类型

- string（但是只支持双引号）
- number（支持科学计数法）
- bool
- null
- object
- array

{% note warning %}
JSON 中，不支持函数和变量
{% endnote %}

#### JSON.parse()

> 反序列化

- 将符合 JSON 语法的字符串转换为 JS 对应类型的数据
- JSON 字符串 ⇒ JS 数据
- 由于 JSON 只有六种类型，所以转换出的数据也只有六种
- 如果不符合 JSON 语法，则会抛出 Error，一般用 try catch 捕获错误，比如

```js
let object
try {
	object = JSON.parse(`{'name':'harvey'}`)
} catch (error) {
	console.log('出错了，错误详情是：')
	console.log(error)
	object = {'name': 'no name'}
}
console.log(object)
```

#### JSON.stringify()

> 序列化

- 是 JSON.parse 的逆运算
- JS 数据 ⇒ JSON 字符串
- 由于 JS 的数据类型比 JSON 多，所以不一定能够成功
- 如果失败，则会抛出一个 Error 对象

## 异步

### 异步与同步

- **同步**： **能直接拿到结果，不拿到结果就不离开**，就像医院挂号一样，可能需要花上几分钟，但是 **拿到号之后才会离开窗口**
- **异步**： **不能直接拿到结果**，比如在餐厅门口等位，拿到号之后我们可以去逛街，那什么时候真正吃上饭呢？我们可以：
  - 每 10 分钟去餐厅问一下（**轮询**）
  - 也可以扫码用微信接收通知（**回调**）

AJAX 举例：当 `request.send()` 之后，并不能直接得到 `response`，必须要等待 `readyState` 变成 `4` 之后，浏览器回头调用 `request.onreadystatechange` 函数

### 回调（callback）

- 写给自己的函数，不是回调
- 写给别人用的函数，放在一个地方，让他来调用，就是回调
- `request.onreadystatechange`，是写给浏览器调用的，意思是让浏览器回头调一下这个函数

举例：

```js
function f1(){}
function f2(fn){
  fn()
}
f2(f1)
// 我调用了 f2，因此 f2 不是回调
// 我把 f1 传给了 f2
// 我没有调用，而是 f2 调用了 f1
// 所以 f1 是回调
```

### 异步与回调

#### 关联

异步任务需要在得到结果时通知 JS 来拿结果，怎么通知呢？

1. 可以让 JS 留一个函数地址（电话号码）给浏览器
2. 异步任务完成之后，浏览器调用该函数的地址（拨打电话）
3. 同时 **把结果作为参数** 传给该函数（电话里说可以来吃了）

这个函数是我写给浏览器调用的，所以是回调函数

#### 区别

异步任务需要用回调函数来通知结果

- 但异步也可以用 **轮询**
- 回调也不一定用在异步里面，比如 `array.forEach( n ⇒ console.log(n) )`，这就是 **同步回调**

### 异步函数

[MDN Async Function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

如果一个函数的返回值处于以下这三个内部，即为异步函数：

- `setTimeout`
- AJAX（即 `XMLHttpRequest`）
- `AddEventListener`

也可以设置同步的 AJAX，`request.open("get", "/5.json", false)`，但是这样会使页面在请求期间卡住

举例：

```js
function 摇骰子(){
  setTimeout( () => {
    return parseInt(Math.random() * 6) + 1
  }, 1000)
  // return undefined
}

// 如果是按上面这样写，将拿不到 1 ~ 6 的随机数，而是 undefined
const n = 摇骰子()
console.log(n) // undefined

// 通过回调来拿到异步的结果
function 摇骰子(fn){
  setTimeout( () => {
    fn(parseInt(Math.random() * 6) + 1)
  }, 1000)
}
function f1(x){ console.log(x) }
摇骰子(f1)

// 可以简化代码，因为 f1 只用了一次
function 摇骰子(fn){
  setTimeout( () => {
    fn(parseInt(Math.random() * 6) +1)
  }, 1000)
}
摇骰子(x => {
  console.log(x)
})

// 可以再简化为
摇骰子(console.log) // 但是如果参数个数不一致就不能这样简化
```

可以再看一下下面这道题：

```js
const array = ['1', '2', '3'].map(parseInt)
  console.log(array) // [1, NaN, NaN]
    
  // 正确的写法，i 和 arr 可以省略
  const array = ['1', '2', '3'].map((item, i, arr) => {
    return paseInt(item)
  })
    
  // 最开始的写法相当于
  const array = ['1', '2', '3'].map((item, i, arr) => {
    return paseInt(item, i, arr)
    // parseInt('1', 0, arr)，0 为无效参数，忽略
    // parseInt('2', 1, arr)，相当于把 '2' 作为 1 进制来进行解析 => NaN
    // parseInt('3', 2, arr)，相当于把 '3' 作为 2 进制来进行解析 => NaN
  })
```

### Promise

> 如果异步任务有两个结果：成功或失败，怎么做？

方法一：回调接受两个参数

```js
fs.readFile('./1.txt', (error, data) => {
  if(error){ console.log('失败'); return }
    console.log(data.toString())
  }
)
```

方法二：使用两个回调

```js
ajax('get', '/1.json', data=>{}, error=>{})
ajax('get', '/1.json', {
  success: ()=>{}, fail: ()=>{}
})
```

但是这两个方法都有问题：

1. 不规范，名称五花八门，有人用 `success + error`，有人用 `success + fail`，有人用 `done + fail`
2. 容易出现回调地狱，代码看不懂
3. 很难进行错误处理

#### 回调地狱

```js
getUser( user => {
  getGroups( user, groups => {
    groups.forEach( (g) => {
      g.filter( x => x.owenerId === user.id)
        .forEach( x => console.log(x))
    })
  })
})
```

#### Promise

##### 基本用法

```js
ajax = (method, url, options) => {
  const {success, fail} = options
  const request = new XMLHttpRequest()
  request.open(method, url)
  request.onreadystatechange = () => {
    if(request.readyState === 4) {
      if(request.status < 400) {
       success.call(null, request.response)
      } else if(request.status >= 400) {
        fail.call(null, request, request.status)
      }
    }
  }
  request.send()
}

ajax('get', '/xxx', {
  success(response){}, fail:(request, status) => {}
})  // 左边是 function 的缩写，右边是箭头函数
```

↑ 上面是普通的写法，↓ 下面是 Promise 的写法

```js
ajax = (method, url) => {
  return new Promise( (resolve, reject) => {  // return new Promise((resolve, reject)=>{..})
    const request = new XMLHttpRequest()
    request.open(method, url)
    request.onreadystatechange = () => {
      if(request.readyState === 4) {
        if(request.status < 400) {
          resolve.call(null, request.response)  // 成功调用 resolve(result)，他再回调用第一个函数
        } else if(request.status >= 400) {
          reject.call(null, request)  // 失败调用 reject(error)，他再会调用第二个函数
        }
      }
    }
    request.send()
  })
}
    
ajax('get', '/xxx')
  .then( (response) => {}, (request) => {} )   // Promise 的回调（成功/失败）只能接受一个参数
```

##### Promise 是如何解决回调地狱的

回调地狱：

1. 多层嵌套
2. 无法方便地进行错误处理

解决办法：

1. 回调函数延迟绑定，回调函数是通过后面的 then 方法传入的
2. 返回值穿透，根据 then 中回调函数的传入值创建不同类型的 Promise，再把返回的 Promise 穿透到外层，以供后续使用。以上两点实现了链式调用，解决多层嵌套问题
3. 错误冒泡，前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了

##### Promise.all()

- 如果传入的参数是一个空的可迭代对象，比如空数组，则返回一个已完成（already resolved）状态的 Promise。
- 如果传入的参数不包含任何 promise，则返回一个异步完成（asynchronously resolved） Promise。
- 其它情况下返回一个处理中（pending）的Promise。

返回的 promise 之后会在所有的 promise 都完成或有一个 promise 失败时异步地变为完成或失败。（如果都成功，则成功；有一个失败，则失败）

返回值将会按照参数内的 promise 顺序排列，而不是由调用 promise 的完成顺序决定。

```js
var p1 = Promise.resolve(3);
var p2 = 1337;
var p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
}); 

Promise.all([p1, p2, p3]).then(values => { 
  console.log(values); // [3, 1337, "foo"] 
});
```

##### Promise.race()

可以传入多个 promise，谁快，Promise.race() 就会跟谁一样，如果传的迭代是空的，则返回的 promise 将永远等待。

# Canvas

## 什么是 Canvas

> `<canvas>` 是 HTML 5 新增的元素，可用于通过使用 JavaScript 的脚本来绘制图形。例如，他可以用于绘制图形、制作照片、创建动画，甚至可以进行实时视频处理和渲染

简单来说，`<canvas>` 提供了一块画布，在上面可以使用 JavaScript 动态渲染图像

### Canvas 与 SVG

> SVG（Scalable Vector Graphics）是基于 XML，用于描述二维矢量图形的一种图片格式，由 W3C 指定，是一个开放标准

Canvas 在绘制结束之后不能被保存到内存中，每次都需要重新绘制，是实时绘制的；svg 则在绘制完之后会被保存在内存中，当需要修改这个对象信息的时候，直接修改就可以了

| Canvas                                   | SVG                                            |
| ---------------------------------------- | ---------------------------------------------- |
| 依赖分辨率（位图）                       | 不依赖分辨率（矢量图）                         |
| 单个 HTML 元素                           | 每个图形都是一个 DOM 元素                      |
| 只能通过脚本语言绘制                     | CSS、脚本语言都可以用来绘制                    |
| 不支持事件处理程序                       | 支持事件处理程序                               |
| 弱的文本渲染恩能够力                     | 最适合带有大型渲染区域的应用程序，比如谷歌地图 |
| 图面较小，对象数量较大时（>10k）性能最佳 | 对象数量较小（<10k）、图面更大时性能更佳       |

## Canvas 的应用场景

1. **绘制图表**，比如 EChats 之类的数据可视化库都是使用 Canvas 实现的
2. **小游戏**，基本上所有的 HTML 5 游戏都是基于 Canvas 开发的，因为不需要借助其他任何插件
3. **活动页面**，比如转转轮、刮奖
4. **特效、背景**

## Canvas API

### 设置宽高

可以通过 HTML 或 JS 来设置，不能通过 CSS 设置，这会导致不可预料的缩放

### 获取 Canvas 对象

```js
canvas.getContext(contextType, contextAttributes)
```

`contextType` 是上下文类型，有这样几种：

- `2d`：代表一个二维渲染上下文
- `webgl` 或 `experimental-webgl`：代表一个三维渲染上下文
- `webgl2` 或 `experimental-webgl2`：代表一个三维渲染上下文，并且只能在浏览器中实现 WebGL 版本 2（OpenGL ES 3.0）

### 绘制路径

| 方法                 | 描述                                                    |
| -------------------- | ------------------------------------------------------- |
| `fill()`             | 填充路径                                                |
| `stroke()`           | 描边                                                    |
| `arc()`              | 创建圆弧                                                |
| `rect()`             | 创建矩形                                                |
| `fillRect()`         | 绘制矩形路径区域                                        |
| `strokeRect()`       | 绘制矩形路径描边                                        |
| `clearRect()`        | 在给定的矩形内清除指定的像素                            |
| `arcTo()`            | 创建两切线之间的弧/曲线                                 |
| `beginPath()`        | 起始一条路径，或重置当前路径                            |
| `moveTo()`           | 把路径移动到画布中的指定点，不创建线条                  |
| `lineTo()`           | 添加一个新点，然后画布中创建从该点到最后指定点的线条    |
| `closePath()`        | 创建从当前点回到起始点的路径                            |
| `clip()`             | 从原始画布剪切任意形状和尺寸的区域                      |
| `quadraticCurveTo()` | 创建二次方贝塞尔曲线                                    |
| `bezierCurveTo()`    | 创建三次方贝塞尔曲线                                    |
| `isPointInPath()`    | 如果指定的点位于当前路径中，则返回 true，否则返回 false |

#### 绘制曲线

##### arc()

```js
context.arc(x, y, sAngle, eAangle, counterCloseWise)
```

`x` 和 `y` 指定了圆心坐标，`sAngle` 和 `eAngle` 分别为起始角和结束角，`counterCloseWise` 设置为 `true` 表示逆时针绘制

#### 绘制直线

```js
moveTo(x, y) // 把路径移动到画布中的指定点，不创建线条
lineTo(x, y) // 添加一个新点，然后在画布中创建从该点到最后指定点的线条
```

{% note warning %}
如果没有 `moveTo`，那么第一次 `lineTo` 就视为 `moveTo`
{% endnote %}

可以给绘制的直线添加样式：

| 样式         | 描述                                     |
| ------------ | ---------------------------------------- |
| `lineCap`    | 设置或返回线条的结束端点样式             |
| `lineJoin`   | 设置或返回两条线相交时，所创建的拐角类型 |
| `lineWidth`  | 设置或返回当前的线条宽度                 |
| `miterLimit` | 设置或返回最大斜接长度                   |

#### 绘制矩形

```js
fillRect(x, y, width, height) // 绘制一个实心矩形
strokeRect(x, y, width, height) // 绘制一个空心矩形
```

#### 颜色、样式和阴影

创建路径之后，可以设置样式属性，再使用 `fill()` 和 `stroke()` 进行填充或描边

| 属性          | 描述                                     |
| ------------- | ---------------------------------------- |
| fillStyle     | 设置或返回用于填充绘画的颜色、渐变或模式 |
| strokeStyle   | 设置或返回用于笔触的颜色、渐变或模式     |
| shadowColor   | 设置或返回用于阴影的颜色                 |
| shadowBlur    | 设置或返回用于阴影的模糊级别             |
| shadowOffsetX | 设置或返回阴影距形状的水平距离           |
| shadowOffsetY | 设置或返回阴影距形状的垂直距离           |

#### 设置渐变

有这样几个方法可以设置渐变：

| 方法                   | 描述                           |
| ---------------------- | ------------------------------ |
| createLinearGradient() | 创建线性渐变（用在画布内容上） |
| createPattern()        | 在指定的方向上重复指定的元素   |
| createRadialGradient() | 环形的渐变（用在画布内容上）   |
| addColorStop()         | 规定渐变对象中的颜色和停止位置 |

主要使用到下面这个方法

```js
const grd = context.createLinerGradient(x0, y0, x1, y1)
grd.addColorStop(stop, color) // stop 是介于 0.0 ~ 1.0 之间的值
```

#### 图形转换

| 方法           | 描述                                           |
| -------------- | ---------------------------------------------- |
| scale()        | 缩放当前绘图至更大或更小                       |
| rotate()       | 旋转当前绘图                                   |
| translate()    | 重新映射画布上的 (0,0) 位置                    |
| transform()    | 替换绘图的当前转换矩阵                         |
| setTransform() | 将当前转换重置为单位矩阵，然后运行 transform() |

注意旋转实际上是对画布旋转，旋转之后所有的都会旋转

#### 图形绘制

```js
drawImage(img, sx, sy, swidth, sheight, x, y, width, height)
```

- `img`：规定要使用的图像、画布或视频
- `sx`：可选。开始剪切的 x 坐标位置
- `sy`：可选。开始剪切的 y 坐标位置
- `swidth`：可选。被剪切图像的宽度
- `sheight`：可选。被剪切图像的高度
- `x`：在画布上放置图像的 x 坐标位置
- `y`：在画布上放置图像的 y 坐标位置
- `width`：可选。要使用的图像的宽度（伸展或缩小图像）
- `height`：可选。要使用的图像的高度（伸展或缩小图像）
