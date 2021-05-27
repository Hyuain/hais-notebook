---
title: JavaScript 数据类型与运算符
date: 2020-01-29 14:55:42
tags:
  - 入门
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

JavaScript 有 7 种数据类型，3 种变量声明的方式，以及一些奇怪的运算符。

<!-- more -->

# 字符存储

- 如何存数字？十进制转二进制，用十六进制（HEX）表示二进制
- 如何存字符？转换为数字，48 号表示 0，65 号表示 A，97 号表示 a
- 如何表示汉字（GB、GBK）？0000~FFFF，16 位，2 字节
- Unicode 已收录 13 万字符（大于 16 位），全世界通用，以后还会继续扩充；缺点：两个字节不够用了，至少要三个字节
- UTF-8：通过变长的存法，减小容量

# 7 种数据类型

> 4 基 2 空 1 对象

## Number

> 64 位浮点数组成

### 特殊值
 
`0` `+0` `-0`
`Infinity` `+Infinity` (1/0) `-Infinity` (1/-0)
`NaN`（0/0，但他还是一个数字，NaN不等于NaN）

### 范围和精度

![JSNumber的存储](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Number-Range.png)

范围：`Number.MAX_VALUE` 到 `Number.MIN_VALUE`
精度：大概是 15 个十进制有效数字

## String

> 每个字符两个字节（阉割版 UTF-8，两个字符定长）

### 写法

单引号、双引号、反引号，引号不是字符串的一部分

### 转义

`\r` 表示回车， `\uFFFF` 表示对应的 Unicode 字符， `\xFF` 表示前256个 Unicode 字符

### 字符串的属性

字符串本来不应该有属性，只有对象才有属性，但是这个有渊源

- 长度： `s.length`
- 下标： `s[0]`
- base64 转码： `window.btoa` 编码， `window.atob` 反编码

## Boolean

> 否定运算、相等运算、比较运算可以得到 bool 值

{% note warning %}
**5 个 falsy 值**
`undefined` `null` `0` `NaN` `''`
{% endnote %}

## Symbol

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

## undefined

如果声明了没有赋值，默认是 `undefined`
如果函数没有写 `return`，默认是 `undefined`

## null

习惯上把非对象空值写成 undefined，对象空值写成 null；
null 通常表示即该处不应该有值，undefined 通常表示"缺少值"，就是此处应该有一个值，但是还没有定义。

## Object

{% note warning %}
数组、函数、日期都是 Object，不是单独的数据类型，但是使用 `typeof` 的时候，可以分辨出 function
{% endnote %}

# 3 种变量声明的方式

## var

- 过时的，不好的
- 函数作用域
- 可以重复声明
- 可以先使用再声明
- 全局声明的 `var` 变量会变成 `window` 的属性

## let

- 新的，更合理的
- 遵循块作用域
- 不能重复声明
- 可以赋值，也可以不赋值
- 必须先声明再使用
- 全局声明的 `let` 变量不会再变成 `window` 的属性
- `let` 配合 `for` 循环有奇效

## const 

跟 `let` 几乎一样，但声明时必须赋值，且不能再更改

{% note warning %}
变量声明指定值的时候同时也指定了类型，但是 **值和类型都可以随意变化**
{% endnote %}

# 运算符

## 算术运算符

### Number

- 余数 `-x % 7` 为 `-(x % 7)`
- 指数 `x ** 3`
- 自增 `a++` 表达式的值是 `a` 加之前的值， `++a` 表达式的值是 `a` 加之后的值
- 求值运算符 `+`，附属运算符 `-`

### String

- 连接运算 `+`

{% note warning %}
`number + string`，变成字符串
`string - number`，变成数字
{% endnote %}

## 比较运算符

### ==

- 模糊相等，发生自动类型转换，别用两个等于
- JavaScript 三位一体

### ===

全等，基本类型看值是否相等，对象看地址是否相等

- `[] !== []`
- `{} !== {}`
- `NaN !== NaN`

## 布尔运算符

```js
console && console.log && console.log('hi') // 防止 console.log 报错（防御性编程）
a = a || 100 // 但是五个 falsy 值都会让 a 为假，因此有 bug

// 可以用函数来赋初值
function add(n = 0) {
  return n + 1
}
```

## 二进制运算符

### 或与否

```js
(0b1111 | 0b1010).toString(2) // 1111
(0b1111 & 0b1010).toString(2) // 1010
(~0b1010).toString // 涉及到补码，留坑
```

### 异或

```js
(0b1111 ^ 0b1010).toString(2) // 101
```

{% note warning %}
按位取反可以用 `^1`
{% endnote %}

### 左移和右移

```js
(0b0011 >> 1).toString(2) // 1
(0b0010 << 1).toString(2) // 100
```

### 头部补零的右移运算符

```js
(0b0011 >>> 1).toString(2) // 1
```

### 如何使用运算符判断奇偶？

```js
偶数 & 1 === 0
奇数 & 1 === 1
```

### 使用 `~` `>>` `<<` `>>>` `|` 来取整

> 位运算不支持小数，会自动抹去

```js
~~ 6.83 // 6
6.83 >> 0
6.83 << 0
6.83 | 0
6.83 >>> 0
```

### 使用 `^` 来交换 `a` `b` 的值

```js
a ^= b
b ^= a
a ^= b

// 新版语法
[a, b] = [b, a]
```

## 其他运算符

### 点运算符

```js
对象.属性名 = 属性值
```

### void 运算符

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

### 逗号运算符

> 表示取后面的值

```js
var a = (1, 2) // a 为 2

let f = x => { console.log('hi'); return x + 1 }
let f = x => (console.log('hi'), x + 1)
// 跟上面是一样的，先执行 console.log，再让 return 为x + 1
```

## 优先级

圆括号的优先级最高

# 数据类型检测

## `typeof` `instanceof` `toString` 的比较

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

## 自己实现一个 `instanceof`

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

## `Object.is` 和 `===`

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
