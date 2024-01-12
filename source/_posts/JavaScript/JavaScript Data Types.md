---
title: JavaScript Data Types
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中一共有 8 种数据类型： 5 基 2 空 1 对象。

由于 JavaScript 是动态类型语言，很难在开发环境中进行编译时类型判断，因为 JavaScript 中变量的类型可以任意改变，并且 JavaScript 中还会进行显式或隐式的[[JavaScript Type Casting|类型转换]] 。此时可以通过 `typeof` 等方法在运行时[[JavaScript Type Checking|检查数据类型]]。

<!-- more -->

# Number

> 64 位浮点数组成

## 特殊值

`0` `+0` `-0`

`Infinity` `+Infinity` (1/0) `-Infinity` (1/-0)

`NaN`（0/0，但他还是一个数字，NaN 不等于 NaN）

## 范围和精度

![JSNumber的存储](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-Number-Range.png)

范围：

- 能准确表示的范围在 `Number.MAX_SAFE_INTEGER` 与 `Number.MIN_SAFE_INTEGER`  之间（- 2^53 + 1 ~ 2^53 - 1）
- 事实上能存储 `Number.MAX_VALUE`  ~ `Number.MIN_VALUE` 的值（±1.7976931348623157 * 10^308），但由于只有 64 bit，所以有些值实际上是近似数

精度：大概是 15 个十进制有效数字

## `Number()` & `new Number()`

- `Number(value)` 会将 `value` 转换为 Number 类型
  - 几乎等价于 `+value`，除了对 Symbol 的处理有所不同
  - 对于对象，会按照 `[@@toPrimitive]()` `valueOf` `toString()` 的顺序调用
- `new Number()` 会创建一个 `Number` 对象，而不是属于基本类型 Number 的值，`new Number(42) !== 42` 但 `new Number(42) == 42`
- 一般不使用 `new Number()`

## `Number.parseFloat()` & `Number.parseInt()` & `parseFloat()` & `parseInt()`

- `parseFloat === Number.parseFloat`
- `parseInt === Number.parseInt`
- `parseFloat()` 会将字符串转换为浮点数
  - 如果不是字符串，会先被强制转换为字符串
  - 字符串开头的空格会被忽略
- `parseInt()` 将字符串转换为整数
  - 接受两个参数，第二个参数是进制
  - 其他与 `parseFloat` 相同

```javascript
const object1 = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      return 42
    }
    if (hint === 'string') {
      return 99
    }
    return null
  }
}

const date1 = new Date("December 17, 1995 03:24:00")

// 下面展示 Number parseInt parseFloat 和 + 的类型转换
// 如果其他没有写出来的，则表示与 Number 结果相同
// 如果 parseFloat 没有写出来，表示与 parseInt 相同

Number(true)                   // 1
parseInt(true)                 // NaN, String(true) === 'true'

Number(false)                  // 0
parseInt(false)                // NaN, String(false) === 'false'

Number(null)                   // 0
parseInt(null)                 // NaN, String(null) === 'null'

Number(undefined)              // NaN
parseInt(undefined)            // NaN, String(undefined) === 'undefined'

Number({})                     // NaN
parseInt({})                   // NaN, String({}) === '[object Object]'

Number(object1)                // 42
parseInt(object1)              // 99

Number(date1)                  // 819199440000
parseInt(date1)                // NaN, String(date1) === 'Sun Dec 17 1995 03:24:00 GMT+0800 (中国标准时间)'

Number(12345678901234567890n)  // 12345678901234567000
+12345678901234567890n         // Cannot convert a BigInt value to a number
parseInt(12345678901234567890n)// 12345678901234567000, String(12345678901234567890n) === '12345678901234567890'

Number(Symbol())               // Cannot convert a Symbol value to a number
parseInt(Symbol())             // Cannot convert a Symbol value to a string

Number("37.12")                // 37.12
parseInt("37.12")              // 37
parseFloat("37.12")            // 37.12

Number("3122d")                // NaN
parseInt("3122d")              // 3122

Number("d3122")                // NaN

Number("37.12.12")             // NaN
parseInt("37.12.12")           // 37
parseFloat("37.12.12")         // 37.12

Number("0x11")                 // 17

Number("0b11")                 // 3
parseInt("0b11")               // 0

Number("0o11")                 // 9
parseInt("0o11")               // 0

Number("")                     // 0
parseInt("")                   // NaN

Number("      ")               // 0
parseInt("      ")             // NaN
```

## `Number.isNaN()` & `isNaN()`

- `Number.isNaN` 更加健壮，他不会进行类型转换，他只对 `NaN` 返回 `true`，与 `x !== x` 等价
- 全局 `isNaN` 会先将值转换为 Number 类型，对所有转换为 Number 为 `NaN` 的，也会返回 `true`

```javascript
isNaN("i'm a string") // true
isNaN(undefined)      // true
isNaN({})             // true
isNaN(true)           // false, Number(true) === 1
isNaN(null)           // false, Number(null) === 0
isNaN("37.12")        // false, Number("37.12") === 37.12
isNaN("3122d")        // true
isNaN("")             // false, Number("") === 0
isNaN("      ")       // false, Number("      ") === 0
```

# String

> 每个字符两个字节（阉割版 UTF-8，两个字符定长）

## 写法

单引号、双引号、反引号，引号不是字符串的一部分

## 转义

`\r` 表示回车， `\uFFFF` 表示对应的 Unicode 字符， `\xFF` 表示前256个 Unicode 字符

## 字符串的属性

字符串本来不应该有属性，只有对象才有属性，见下面的解释

- 长度： `s.length`
- 下标： `s[0]`
- base64 转码： `window.btoa` 编码， `window.atob` 反编码

## `String()` & `new String()`

- `String()` 得到的是一个字符串基本类型
- `new String()` 得到的是一个对象
- 会发生强制类型转换，值得注意的是：
  - `undefined` `null` 都会被转换成那几个英文字母
  - 对于对象或其他非空的基本类型，会按照 `[@@toPrimitive]()` `toString()` `valueOf` 的顺序调用
  
## String 基本类型与 String 对象

>  Boolean 和 Number 有同样的问题，都有基本类型和对象的区别

- String 基本类型可以通过字面量（单双引号）或者 `String()` 的返回值得到
- String 对象可以通过 `new String()` 获得，但通常不怎么使用
- JavaScript 会自动检查访问基本类型 String 属性、或者在基本类型上调用方法的地方，并自动将其包裹成一个对象（Auto-Boxing），访问对象上的属性或方法
- 他们在传给 `eval()` 的时候表现也有所不同：String 基本类型会被当成 JavaScript 源码，而 String 对象会被当做一个对象

# Boolean

> 否定运算、相等运算、比较运算可以得到 bool 值

{% note warning %}

**6 个 falsy 值**

`undefined` `null` `0` `NaN` `''` `document.all`

{% endnote %}

# Symbol

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

new Symbol(123) // TypeError: Symbol is not a constructor
```

# BigInt

> BigInt 可以准确表示任何大小的整数值

```javascript
// the "n" at the end means it's a BigInt
const bigInt = 1234567890123456789012345678901234567890n;
```

# undefined

如果声明了没有赋值，默认是 `undefined`

如果函数没有写 `return`，默认是 `undefined`

# Null

习惯上把非对象空值写成 undefined，对象空值写成 null；

null 通常表示即该处不应该有值，undefined 通常表示"缺少值"，就是此处应该有一个值，但是还没有定义。

# Object

{% note warning %}

数组、函数、日期都是 Object，不是单独的数据类型，但是使用 `typeof` 的时候，可以分辨出 function

{% endnote %}

# `toPrimitive` & `valueOf` & `toString`

<table>
  <tr>
    <td colspan="2">类型</td>
    <td>举例</td>
    <td>@@toPrimitive</td>
    <td>valueOf</td>
    <td>toString</td>
  </tr>
  <tr>
    <td rowspan="5">基本类型</td>
    <td>Number</td>
    <td>1</td>
    <td>-</td>
    <td>返回数字基本类型</td>
    <td>
      <p>可以接受一个参数，表示进制</p>
      <p>注意数字基本类型发生强制类型转换时（比如 `${someNumber}`），并不会调用 Number.prototype.toString()，所以覆盖这个方法并不会影响到此种情况的类型转换结果</p>
    </td>
  </tr>
  <tr>
    <td>String</td>
    <td>'hello'</td>
    <td>-</td>
    <td>返回字符串基本类型</td>
    <td>与 valueOf 实现完全相同</td>
  </tr>
  <tr>
    <td>Boolean</td>
    <td>true</td>
    <td>-</td>
    <td>返回布尔基本类型</td>
    <td>返回字符串 "true" "false"</td>
  </tr>
  <tr>
    <td>Symbol</td>
    <td>Symbol()</td>
    <td>*1</td>
    <td>与 toPrimitive 效果相同</td>
    <td>*2</td>
  </tr>
  <tr>
    <td>BigInt</td>
    <td>12345678n</td>
    <td>-</td>
    <td>返回 BigInt 基本类型</td>
    <td>返回一个去掉 n 的字符串</td>
  </tr>
  <tr>
    <td rowspan="6">对象类型</td>
    <td>普通对象</td>
    <td>{a: '1', b: '2'}</td>
    <td>默认没有，可以自己定义</td>
    <td>对象自己，这是无意义的，因为 valueOf() 一般是用来转换成基本类型的，因此需要自己定义 valueOf</td>
    <td>根据 this 返回 "[object Type]"</td>
  </tr>
  <tr>
    <td>Array</td>
    <td>[1, 2, 3]</td>
    <td>-</td>
    <td>-</td>
    <td>调用 Array.prototype.join</td>
  </tr>
  <tr>
    <td>Function</td>
    <td>function() {}</td>
    <td>-</td>
    <td>-</td>
    <td>根据 this 返回函数字符串</td>
  </tr>
  <tr>
    <td>Error</td>
    <td>new Error()</td>
    <td>-</td>
    <td>-</td>
    <td>返回错误信息</td>
  </tr>
  <tr>
    <td>Date</td>
    <td>new Date()</td>
    <td>*3</td>
    <td>与 Date.prototype.getTime 效果相同</td>
    <td>返回日期字符串</td>
  </tr>
  <tr>
    <td>RegExp</td>
    <td>new RegExp()</td>
    <td>-</td>
    <td>-</td>
    <td>*4</td>
  </tr>
</table>

*1. `Symbol.prototype[@@toPrimitive]` 会返回一个 Symbol 基本类型，其中的 `hint` 参数并没有被用到：

```js
const sym = Symbol("example")
sym === sym[Symbol.toPrimitive](); // true
```

*2. `Symbol.prototype.toString` 需要提供一个 `this`，这个 `this` 应该是一个 Symbol 基本类型或 Symbol 对象。如果不提供正确的 `this`，它将会抛错，因为 `Symbol.prototype[@@toPrimitive]` 方法只会返回一个 Symbol 基本类型：

```js
console.log(Symbol('desc').toString());
// Expected output: "Symbol(desc)"

console.log(Symbol.iterator.toString());
// Expected output: "Symbol(Symbol.iterator)

console.log(Symbol.for('foo').toString());
// Expected output: "Symbol(foo)"

// console.log(Symbol('foo') + 'bar');
// Expected output: Error: Can't convert symbol to string
```

*3. `Date.prototype[@@toPrimitive]` 会根据参数返回时间戳或者字符串：

```js
// Depending on timezone, your results will vary
const date = new Date('20 December 2019 14:48');

console.log(date[Symbol.toPrimitive]('string'));
// "Fri Dec 20 2019 14:48:00 GMT+0800 (中国标准时间)"

console.log(date[Symbol.toPrimitive]('number'));
// 1576824480000
```

*4. `RegExp.prototype.toString` 会返回类似 `/source/flag` 的字符串：

```js
console.log(new RegExp('a+b+c'));
// "/a+b+c/

console.log(new RegExp('a+b+c').toString());
// "/a+b+c/"

console.log(new RegExp('bar', 'g').toString());
// "/bar/g"

console.log(new RegExp('\n', 'g').toString());
// "/\n/g" (if your browser supports escaping)

console.log(new RegExp('\\n', 'g').toString());
// "/\n/g"
```
