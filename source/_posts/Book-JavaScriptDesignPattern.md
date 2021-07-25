---                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     
title: 读书笔记：《JavaScript 设计模式与开发实践》
date: 2021-07-17 20:06:11
tags:
  - 读书笔记
categories:
    - [前端, JavaScript, 原生 JavaScript]
---          

妄图了解设计模式，开始阅读此书。

<!-- more -->

# 基础知识

## 面向对象的 JavaScript

### 动态类型

> 动态类型语言的变量类型要到程序运行时才能确定，比如 JavaScript

- Duck Typing 鸭子类型：如果他走起来像鸭子，叫起来也是鸭子，那么他就是鸭子
- 面向接口编程：在动态类型语言中，我们可以很轻松地实现”面向接口编程，而不是面向实现编程“。比如若一个对象有 `pop` 和 `push` 方法，那么他就可以被当做栈来使用

### 多态

> 同一个操作作用于不同的对象上面，可以产生不同的解释和执行结果。比如让鸭子和鸡都执行“叫”命令，他们会按照自己的方式叫出不同的叫声

- 可以使用继承得到多态的效果：鸡和鸭都分别继承“动物“抽象类，动物类定义了”叫“ 
- 多态在面向对象设计中的作用：可以将过程化的条件分支语句转换为对象的多态性，从而减少这种条件分支语句

### 封装

将信息隐藏起来，可以封装数据、实现、类型、变化等

#### 封装数据

有的语言通过语法解析来实现，同时提供了 `private` `public` `protected` 等关键字确定访问权限

JavaScript 中可以通过变量的作用域来实现封装，书中介绍了基于 `var` 的函数作用于特性的模拟 `public` 和 `private` 的方法，在 ES6 中同样可以通过闭包达到类似的封装效果

#### 封装实现

比较常见的封装，封装一些功能，然后提供 API 给外界使用

#### 封装类型

在静态类型语言中经常涉及到封装类型：通常是通过提供抽象类和接口来实现的。

#### 封装变化

把系统中稳定不变的部分和容易变化的部分隔离开来

## `this` `call` `apply`

TODO: 以后再详细读这个部分

## 闭包和高阶函数

TODO：以后再详细读这个部分

# 设计模式

## 单例模式

> 保证一个类仅有一个实例，并提供一个访问他的全局访问点

### 简单的单例模式

通过 `Singleton.getInstance` 来获取 `Singleton` 类的唯一对象，但增加了这个类的不透明性：不能通过 `new` 直接来构造这个对象，需要通过对象提供的 `getInstance` 方法。

```javascript
const Singleton = function (name) {
  this.name = name
}
Singleton.getInstance = function (name) {
  if (!this.instance) {
    this.instance = new Singleton(name)
  }
  return this.instance
}
```

### 透明的单例模式

用户可以直接使用 `new` 构造一个对象，并且不管 `new` 多少次，得到的都是全局唯一的单例。这里使用了自执行函数和闭包

```javascript
const CreateDiv = (function () {
  let instance
  const RealCreateDiv = function (html) {
    if (instance) {
      return instance
    }
    this.html = html
    this.init()
    return instance = this
  }
  RealCreateDiv.prototype.init = function () {
    const div = document.createElement('div')
    div.innerHTML = this.html
    document.body.appendChild(div)
  }
  return RealCreateDiv
})()

const a = new CreateDiv('1')
const b = new CreateDiv('2')
console.log(a === b) // true
```

缺点是复杂，并且如果有一天我们想要用这个类构造出多例，就得深入业务代码去改

### 利用代理实现单例模式

我们可以将实现单例的代码给抽离出来

```javascript
// 先写一个普通的构造函数
const CreateDiv = function (html) {
  this.html = html
  this.init()
}
CreateDiv.prototype.init = function () {
  const div = document.createElement('div')
  div.innerHTML = this.html
  document.body.appendChild(div)
}

// 再引入代理类 proxySingletonCreateDiv
const ProxySingletonCreateDiv = (function () {
  let instance
    return function (html) {
      if (!instance) {
        instance = new CreateDiv(html)
      }
      return instance
    }
})()

const a = new ProxySingletonCreateDiv('1')
const b = new ProxySingletonCreateDiv('2')
console.log(a === b) // true
```

### 惰性单例
在需要的时候才创建出单例

```javascript
const getSingle = function (fn) {
  let result
  return function () {
    return result || ( result = fn.apply(this, arguments ) )
  }
}
```

## 策略模式

定义一些列算法，把他们一个个封装起来，并且使他们可以相互替换

### 使用策略模式计算奖金

```javascript
const calculateBonus = function (performanceLevel, salary) {
  if (performanceLevel === 'S') return salary * 4
  if (performanceLevel === 'A') return salary * 3
  if (performanceLevel === 'B') return salary * 2
}
```

`calculateBonus` 函数非常庞大且缺乏可扩展性、复用性

基于策略模式的程序至少包含两部分：

- 一组策略类：策略类封装了具体的算法，并负责具体的计算过程
- 环境类 Context：接受客户请求，然后把请求委托给一个策略类

```javascript
const performanceS = function (){}
performanceS.prototype.calculate = function (salary) {
  return salary * 4
}
const performanceA = function (){}
performanceA.prototype.calculate = function (salary) {
  return salary * 3
}
const performanceB = function (){}
performanceB.prototype.calculate = function (salary) {
  return salary * 2
}

const Bonus = function () {
  this.salary = null
  this.strategy = null
}
Bonus.prototype.setSalary = function (salary) {
  this.salary = salary
}
Bonus.prototype.setStrategy = function (strategy) {
  this.strategy = strategy
}
Bonus.prototype.getBonus = function () {
  return this.strategy.calculate(this.salary)
}
```

也可以这样写让代码更简洁：

```javascript
const strategies = {
  "S": function (salary) {
      return salary * 4
  },
  "A": function (salary) {
      return salary * 3
  },
  "B": function (salary) {
      return salary * 2
  },
}

const calculateBonus = (level, salary) => {
  return strategies[level](salary)
}
```