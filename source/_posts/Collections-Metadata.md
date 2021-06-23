---
title: 元编程
date: 2021-06-07 21:55:48
tags:
  - 入门
  - 这一秒是你的，下一秒就是我的
categories:
  - [前端, 收集]
---

抄袭自腾讯 VTeam 技术团队的文章《前端元编程——使用注解加速你的前端开发》。

<!-- more -->

尝试用 Decorator 和 Reflect 元编程来减少模板代码。 

# 元编程

## Decorator

Decorator 可以帮助我们修改类和他们的成员。

```typescript
// offer type
abstract class Base {
  log() {}  
}

function EnhanceClass() {
  return function (Target) {
    return class extends Target {
      log() {
        console.log("---log---")
      }
    }
  }
}

@EnhanceClass()
class Person extends Base { }

new Person().log() // ---log---
```

## Reflect

Reflect 是一个 ES6 中的内置对象，提供了拦截 JavaScript 操作的方法。Vue3 中就依赖 Reflect 和 Proxy 来重写了响应式逻辑。

```typescript
const _list = [ 1, 2, 3 ]
const pList = new Proxy(_list, {
  get(target: number[], p: string | symbol, receiver: any): any {
    console.log("get value reflect: ", p)
    return Reflect.get(target, p, receiver)
  },
  set(target: number[], p: string | symbol, value: any, receiver: any): boolean {
    console.log("set value reflect: ", p, value)
    return Reflect.set(target, p, value, receiver)
  }
})
pList.push(4)
// get value reflect: push
// get value reflect: length
// set value reflect: 3 4
// set value reflect: length 4
```

## Reflect Metadata

Reflect Metadata 能够为对象添加和读取元数据。

```typescript
function Type(): PropertyDecorator {
  return function (target, propertyKey) {
    const type = Reflect.getMetadata("design:type", target, propertyKey)
    console.log(propertyKey, " type: ", type)
  }
}

class Person extends Base {
  @Type()
  name: string = ''
}

// name type: String
```

# 用元编程减少 CURD 模板代码

首先需要一个函数来生成不同业务的属性装饰器：

```typescript
function CreateProperDecoratorF<T>() {
  const metaKey = Symbol()
  function properDecoratorF(config: T): PropertyDecorator {
    return function (target, propertyKey) {
      Reflect.defineMetadata(metaKey, config, target, propertyKey)
    }
  }
  return { metaKey, properDecoratorF }
}
```

一个用来处理通过数据装饰器收集上来的元数据的类装饰器：

```typescript
function EnhancedClass(config: ClassConfig) {
  return function (Target) {
    return class extends Target {
    }
  }
}
```

## 栗子：API Model 映射

将后端数据转换为 type, interface 或者 class，这里选择了在编译后的 JavaScript 中仍然存在的 class：

```typescript
export interface TypePropertyConfig {
  handle?: string | ServerHandle
}

const typeConfig = CreateProperDecoratorF<TypePropertyConfig>()
export const Type = typeConfig.properDecoratorF

@EnhancedClass({})
export class Person extends Base {
  static sexOptions = [ 'male', 'female', 'unknown' ]
  
  // 相当于每个属性都加了一个 metadata，key 是 CreateProperDecoratorF 中的 symbol（也就是 typeConfig.metaKey），value 是传进去的 { handle: 'ID' }
  @Type({
    handle: 'ID'
  })
  id: number = 0
  
  @Type({})
  name: string = ''
  
  @Type({
    handle(data, key) {
      return parseInt(data[key] || '0')
    }
  })
  age: number = 0
  
  @Type({
    handle(data, key) {
      return Person.sexOptions.includes(data[key] ? data[key] : 'unknown')
    }
  })
  sex: 'male' | 'female' | 'unknown'
}

// 在 EnhancedClass 这个 Decorator 中，拿出上面定义的每一个属性中的 metadata，按照其中 handle 定义的方法对数据进行转换
function EnhancedClass(config: ClassConfig) {
  return function (Target) {
    return class extends Target {
      constructor(data) {
        super(data)
        Object.keys(this).forEach((key) => {
          const config: TypePropertyConfig = Reflect.getMetadata(typeConfig.metaKey, this, key)
          // 将后端数据（data）转换为前端数据（this）
          this[key] = config.handle
            ? typeof config.handle === 'string'
              ? data[config.handle]
              : config.handle(data, key)
            : data[key]
        })
      }
    }
  }
}
```

## 栗子：列表页 TablePage

原文中举了 Tea Component 的例子，通过类似的操作收集到 