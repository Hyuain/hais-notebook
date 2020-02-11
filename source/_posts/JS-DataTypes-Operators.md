---
title: JavaScript 数据类型与运算符
date: 2020-01-29 14:55:42
tags:
  - 入门
  - 饥人谷
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

![](/hais-notebook/images/JS-001.png)

范围：`Number.MAX_VALUE` 到 `Number.MIN_VALUE`
精度：大概是 15 个十进制有效数字

## String

> 每个字符两个字节（阉割版 UTF-8，两个字符定长）

### 写法

单引号、双引号、反引号，引号不是字符串的一部分

### 转义

`\r` 表示回车， `\uFFFF` 表示对应的 Unicode 字符， `\xFF` 表示前256个 Unicode 字符

### 字符串的属性

字符串本来不应该有属性，只有对象才有属性，但是这个有渊源

- 长度： `s.length`
- 下标： `s[0]`
- base64 转码： `window.btoa` 编码， `window.atob` 反编码

## Bool

> 否定运算、相等运算、比较运算可以得到 bool 值

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
- 全局声明的 `let` 变量不会再变成 `window` 的属性
- `let` 配合 `for` 循环有奇效

## const 

跟 `let` 几乎一样，但声明时必须赋值，且不能再更改

{% note warning %}
变量声明指定值的时候同时也指定了类型，但是 **值和类型都可以随意变化**
{% endnote %}

# 运算符

## 算术运算符

### Number

- 余数 `-x % 7` 为 `-(x % 7)`
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
- JavaScript 三位一体

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
(~0b1010).toString // 涉及到补码，留坑
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

### 使用 `^` 来交换 `a` `b` 的值

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
对象.属性名 = 属性值
```

如果不是对象，JS会创建一个对象，用完之后再自动删除

```js
a.xxx = 'harvey' // 'harvey'
a.xxx // undefiend
```

### void 运算符

> 求表达式的值或执行语句，然后 `void` 的值总为 `undefined`

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
var a = (1, 2) // a 为 2

let f = x => { console.log('hi'); return x + 1 }
let f = x => (console.log('hi'), x + 1)
// 跟上面是一样的，先执行 console.log，再让 return 为x + 1
```

## 优先级

圆括号的优先级最高