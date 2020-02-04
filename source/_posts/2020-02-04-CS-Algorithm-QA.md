---
title: 算法小问题：数组去重
date: 2020-02-04 14:51:49
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [计算机, 算法]
---

总结：恰当地利用对象和 Map key 的唯一性可以实现对 `'1'` 和 `1` 的区分、对象的去重，以及 `undefined` `null` `NaN` 的良好识别。

<!-- more -->

使用数组 `[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]` 输出进行实验，其中：

```js
const x = { b: 1 }
const y = { b: 1 }
const z = { a: 1 }
```

# 双重循环

```js
Array.prototype.unique = function() {
  const newArray = []
  let isRepeat
  for (let i = 0; i < this.length; i++) {
    isRepeat = false
    for (let j = 0; j < newArray.length; j++) {
      if(this[i] === newArray[j]){
        isRepeat = true
        break
      }
    }
    if (!isRepeat) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

# `indexOf`

```js
Array.prototype.unique = function() {
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    const current = this[i]
    if (newArray.indexOf(current) < 0) {
      newArray.push(current)
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

# 相邻元素去重（先排序）

```js
Array.prototype.unique = function() {
  const newArray = []
  this.sort()
  for (let i = 0; i < this.length; i++) {
    if(this[i] !== this[i+1]) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`，`undefined`（会丢失）

# `filter`

```js
Array.prototype.unique = function() {
  return newArray = this.filter((item, index, array) => {
    return array.indexOf(item) === index
  })
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null]
```

不能区分：对象，`NaN`（会丢失）

# `reduce`（先排序）

```js
Array.prototype.unique = function() {
  return this.sort().reduce((result, current) => {
    if (result.length === 0 || result[result.length - 1] !== current) {
      result.push(current)
    }
    return result
  }, [])
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null, undefined]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`

# `includes`

```js
Array.prototype.unique = function() {
  const newArray = []
  this.forEach(item => {
    if (!newArray.includes(item)) {
      newArray.push(item)
    }
  })
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

注意：原来的数组会被更改！

不能区分：对象

# 对象键值对

利用对象中 key 的不可重复性去重

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[this[i]]) {
      obj[this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, 0, { a: 1 }, undefined, null, NaN]
```

不能区分：对象（存进去的时候所有对象都变成了 `[object Object]`）、'1' 和 1

## 改进版本 1

可以区分 `'1'` 和 `1`

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[typeof this[i] + this[i]]) {
      obj[typeof this[i] + this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

## 改进版本 2

可以区分对象

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if(!obj[typeof this[i] + JSON.stringify(this[i])]) {
      obj[typeof this[i] + JSON.stringify(this[i])] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { b: 1 }, undefined, null, NaN]
```

# Map

利用 Map 的 key 唯一

```js
Array.prototype.unique = function() {
  const newArray = []
  const map = new Map()
  for (let i = 0; i < this.length; i++) {
    if(!map.get(this[i])) {
      map.set(this[i], true)
      newArray.push(this[i])
    }
  }
  return newArray
}

// 简化版
Array.prototype.unique = function() {
  const map = new Map()
  return this.filter((item) => (!map.has(item) && map.set(item, true)))
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象，可以像上面的对象键值对方法一样处理

# Set

Set 类似于数组，但成员的值都是唯一的

```js
Array.prototype.unique = function() {
  return Array.from(new Set(this))
}

// 可以简化为
Array.prototype.unique = function() {
  return [...new Set(this)]
}

// 还可以简化为
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象