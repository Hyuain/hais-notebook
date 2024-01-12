---
title: JavaScript Deep Clone
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中对象类型的直接赋值只是复制了其堆中的引用的地址，有时候需要对对象进行深拷贝（Deep Clone）。

<!-- more -->

```js
const obj = {
  a: 100,
  b: [10, 20, 30],
  c: {
    x: 10
  },
  d: /^\d+$/
}
```

# 直接赋值

```js
const obj2 = obj1
```

当改变 `obj1` 的时候，`obj2` 会跟着改变，这是直接赋值，不属于深/浅克隆

# 浅克隆

> 只克隆第一层

## 手动实现

```js
const obj2 = {}
for(let key in obj) {
  if(!obj.hasOwnProperty(key)) continue // 这里也可以用 break，因为到这一步基本就已经到原型的属性了，可以直接跳出循环
  obj2[key] = obj[key]
}
```

## Object.assign

他拷贝的是对象属性的引用，而不是对象本身

```js
let obj2 = Object.assign({}, obj)
```

## 展开运算符

```js
const obj2 = { ...obj }
```

## concat

```js
let arr = [1, 2, 3]
let newArr = arr.concat()
```

## slice

```js
let arr = [1, 2, 3]
let newArr = arr.slice()
```

# 深克隆

> 每一层都克隆

## 简易版的深拷贝

最最最简单的深拷贝可以这样写：

```js
const obj2 = JSON.parse(JSON.stringify(obj))
```

但是在有些时候是会出现问题的：

1. 循环引用
2. 无法处理 RegExp、Date、Set、Map、Function 等

也可以用 `lodash` 之类的库，当然也可以手写一个简易版的

```js
function deepClone(target) {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? [] : {}
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop])
      }
    }
    return cloneTarget
  } else {
    return target
  }
}
```

## 解决循环引用问题

思路：创建一个 Map，将已经拷贝过的对象记录下来，如果发现已经拷贝过，就直接返回他

```js
const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null

const deepClone = (target, map = new weakMap()) => {
  if (map.get(target)) {
    return target
  }
  
  if (isObject(target)) {
    map.set(target, true)
    const cloneTarget = Array.isArray(target) ? [] : {}
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
        cloneTarget[prop] = deepClone(target[prop], map)
      }
    }
    return cloneTarget
  } else {
    return target
  }
}
```

如果使用 Map，那么 map 的 key 将会与 map 形成强引用，如果强引用一直存在，那么对象将无法被回收；

使用 WeakMap 可以构造一种弱引用（一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收），WeakMap 的 Key 必须是对象，而值可以是任意的。

## 拷贝特殊对象

### 可继续遍历

比如 Map、Set、Array、Object、Arguments：

```js
const getType = Object.prototype.toString.call
const isObject = (target) => (typeof target === 'object' || typeof target === 'function') && target !== null

const canTraverse = {
  '[object Map]': true,
  '[object Set]': true,
  '[object Array]': true,
  '[object Object]': true,
  '[object Arguments]': true
}

const deepClone = (target, map = new WeakMap()) => {
  if (!isObject(target)) return target
  let type = getType(target)
  let cloneTarget
  if (!canTraverse[type]) {
    // 处理不可遍历对象
  } else {
    cloneTarget = new target.constructor()
  }
  
  if (map.get(target)) return target
  map.set(target, true)

  if (type === '[object Map]') {
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key), deepClone(item))
    })
  }

  if (type === '[object Set]') {
    target.forEach(item => {
      target.add(deepClone(item))
    })
  }


  // 处理 Object 和 Array
  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop])
    }
  }
  return cloneTarget
}
```

### 不可遍历

```js
const boolTag = '[object Boolean]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const regExpTag = '[object RegExp]';
const funcTag = '[object Function]';

const handleRegExp = (target) => {
  const {source, flags} = target
  return new target.constructor(source, flags)
}

// 需要区分处理箭头函数和普通函数，因为普通函数是 Function 的实例，而箭头函数不是任何类的实例
const handleFunc = (func) => {
  // 箭头函数返回自身
  if (!func.prototype) return func
  const bodyReg = /(?<={)(.|\n)(?=})/m
  const paramReg = /(?<=\().+(?=\)\s*{)/
  const funcString = func.toString()
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].slice(',')
    return new Function(...param, body[0])
  } else {
    return new Function(body[0])
  }
}

const handleNotTraverse = (target, tag) => {
  const constructor = target.constructor
  switch (tag) {
    case boolTag:
      // 因为用 new Boolean 创造出来的对象有问题
      return new Object(Boolean.prototype.valueOf.call(target))
    case numberTag:
      return new Object(Number.prototype.valueOf.call(target))
    case stringTag:
      return new Object(String.prototype.valueOf.call(target))
    case errorTag:
    case dateTag:
      return new constructor(target)
    case regExpTag:
      return handleRegExp(target)
    case funcTag:
      return handleFunc(target)
    default:
      return new constructor(target)
  }
}
```

## 完整代码

```js
const getType = target => Object.prototype.toString.call(target).match(/\[object (.*?)\]/)[1].toLowerCase()

const isObject = target => (typeof target === 'object' || typeof target === 'function') && target !== null

const canTraverse = {
  'map': true,
  'set': true,
  'array': true,
  'object': true,
  'arguments': true
}

const handleRegExp = target => {
  const {source,flags} = target
  return new target.constructor(source, flags)
}

const handleFunc = func => {
  if (!func.prototype) return func
  const bodyReg = /(?<={)(.|\n)(?=})/m
  const paramReg = /(?<=\().+(?=\)\s+{)/
  const funcString = func.toString()
  const param = paramReg.exec(funcString)
  const body = bodyReg.exec(funcString)
  if (!body) return null
  if (param) {
    const paramArr = param[0].split(',')
    return new Function(...paramArr, body[0])
  } else {
    return new Function(body[0])
  }
}

const handleNotTraverse = (target, type) => {
  const constructor = target.constructor
  switch (type) {
    case 'boolean':
      return new Object(Boolean.prototype.valueOf.call(target))
    case 'number':
      return new Object(Number.prototype.valueOf.call(target))
    case 'string':
      return new Object(String.prototype.valueOf.call(target))
    case 'symbol':
      return new Object(Symbol.prototype.valueOf.call(target))
    case 'error':
    case 'date':
      return new constructor(target)
    case 'regexp':
      return handleRegExp(target)
    case 'function':
      return handleFunc(target)
    default:
      return new constructor(target)
  }
}

const deepClone = (target, map = new WeakMap()) => {
  if (!isObject(target)) {
    return target
  }
  const type = getType(target)
  let cloneTarget
  if (!canTraverse[type]) {
    return handleNotTraverse(target, type)
  } else {
    cloneTarget = new target.constructor()
  }

  if (map.get(target)) return target
  map.set(target, true)

  if (type === 'map') {
    target.forEach((item, key) => {
      cloneTarget.set(deepClone(key, map), deepClone(item, map))
    })
  }
  
  if (type === 'set') {
    target.forEach(item => {
      cloneTarget.add(deepClone(item, map))
    })
  }

  for (let prop in target) {
    if (target.hasOwnProperty(prop)) {
      cloneTarget[prop] = deepClone(target[prop], map)
    }
  }
  return cloneTarget
}
```
