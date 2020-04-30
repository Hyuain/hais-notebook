---
title: TypeScript：进阶
date: 2020-03-21 17:25:49
tags:
  - 入门
categories:
  - [前端, TypeScript]
---

TypeScript 的高级类型以及其他的用法。

<!-- more -->

# 高级类型

## 交叉类型

交叉类型是将多个类型合并为一个类型，比较常见的是用在 JavaScript 的混入模式（从两个对象中刚创建一个新对象）中：

```typescript
interface IAnyObject {
  [prop: string]: any
}
function mixin<T extends IAnyObject, U extends IAnyObject>(first: T, second: U): T & U {
  const result = <T & U>{};
  for (let id in first) {
    (<T>result)[id] = first[id];
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (<U>result)[id] = second[id];
    }
  } 
  return result;
}

const x = mixin({a: 'hello'}, {b: 42});

const a = x.a;
const b = x.b;
```

## 联合类型

```typescript
function formatCommandline(command: string[] | string) {
  let line = '';
  if (typeof command === 'string') {
    line = command.trim();
  } else {
    line = command.join(' ').trim();
  }
}
```

## 类型别名

```typescript
type a = boolean | string
type Container<T> = {value: T}
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
}
```

{% note warning %}
**type 与 interface？**
- interface 只能用于定义 **对象类型**
- type 还可以定义 **原始类型**、**交叉类型** 和 **联合类型** 等，用途更加广泛
- 但是 interface 可以实现 extends 和 implements，也可以实现接口合并声明
{% endnote %}

# 可辨识联合类型

## 字面量类型

字面量类型（Literal Type）主要分为 **真值字面量类型（Boolean Literal Type）**、**数字字面量类型（Numeric Literal Type）**、**枚举字面量类型（Enum Literal Type）**、**大整数字面量类型（BigInt Literal Type）** 和 **字符串字面量类型（String Literal Type）**

```typescript
const a: 'xiaoming' = 'xiaoming';
const b: 'xiaoming' = 'xiaohong'; // Initializer type "xiaohong" is not assignable to variable type "xiaoming"
```

字面量类型有时候可以模拟一个类似枚举的效果：

```typescript
type Direction = 'North' | 'East' | 'South' | 'West'
```

## 类型字面量

类型字面量（Type Literal）则跟 JavaScript 中的对象字面量语法很相似：

```typescript
type Foo = {
  baz: [
    number,
    'xiaoming'
  ];
  toString(): string
}
```

## 可辨识的联合类型

有时候会出现这样的需求，有两个 `action`，一个是创建用户 `create`，一个是删除用户 `delete`，我们在创建用户的时候不需要 id，在删除用糊的时候需要，因此我们可能会这样写：

```typescript
interface Info {
  username: string
}
interface UserAction {
  id?: number
  action: 'create' | 'delete'
  info: Info
}
```

但是这样并不能区分开创建用户和删除用户操作，我们就需要使用类型字面量

```typescript
type UserAction = {
  id: number
  action: 'delete'
  info: Info
} | {
  action: 'create'
  info: Info
}

const userReducer = (userAction: UserAction) => {
  switch (userAction.action) {
    case 'delete':
      console.log(userAction.id);
      break;
    default:
      break;
  }
}
```

我们可以发现在输入 `userAction.` 之后，IDE 会有所提示，因为我们通过 `userAction.action` 分开了不同的情况，`userAction.action` 就是我们识别的关键，被称为 **可辨识的标签**。

要想达成这样的效果，需要实现这样几个要素：

- 具有普通的单例类型属性，也就是要有可辨识的特征，比如 `delete` 和 `create`
- 一个包含 **联合类型** 的类型别名
- 类型守卫的特征，比如使用 `if` 或 `switch` 等来进行判断

# 装饰器

装饰器的主要作用是给已有的方法或类扩展一些新的行为，而不是修改他本身

需要在 `tsconfig.json` 中添加一些选项使其支持装饰器：

```json
"experimentalDecorators": true
```

装饰器本质上是一个函数，`@expression` 的形式其实是一个语法糖，`expression` 求值后也必须是一个函数，他会在运行时被调用，被装饰的声明信息作为参数传入

## 类装饰器

```typescript
function addAge(constructor: Function) {
  constructor.prototype.age = 18
}

@addAge
class Person {
  name: string;
  age!: number;
  constructor() {
    this.name = 'xiaoming'
  }
}

let person = new Person();

console.log(person.age); // 18 
```

其中 `constructor` 是 `Person`

## 属性/方法装饰器

```typescript
function method(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(target);
  console.log('prop' + propertyKey);
  console.log('desc' + JSON.stringify(descriptor) + '\n\n');
  descriptor.writable = false;
}

class Person {
  name: string;
  constructor() {
    this.name = 'xiaoming';
  }
  
  @method // 其实是相当于使用了 Object.defineProperty 来修改方法和属性
  say() {
    return 'instance method';
  }

  @method
  static run() {
    return 'static method';
  }
}

const xm = new Person();
xm.say = function() { // Cannot assign to read only property 'say' of object '#<Person>'
  return 'edit'
}
```

其中 `target` 是当前对象的原型 `Person.prototype`

## 参数装饰器

参数装饰器在 Angular 和 Nestjs 中都有运用

```typescript
function logParameter(target: Object, propertyKey: string, index: number) {
  console.log(target, propertyKey, index);
}

class Person {
  greet(@logParameter message: string, @logParameter name:string): string {
    return `${message} ${name}`;
  }
}

const p = new Person();
p.greet('hi', 'xiaoming')
```

其中 `target` 是当前对象的原型 `Person.prototype`，`propertyKey` 是 `greet`，在这里装饰器起到的作用主要是能够提供一系列的信息（元数据）

## 装饰器工厂

可以使用装饰器工厂封装不同的装饰器：

```typescript
function logClass(target: Function) {
  console.log(target);
}
function logProperty(target: any, propertyKey: string) {
  console.log(propertyKey);
}
function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(propertyKey);
}
function logParameter(target: Object, propertyKey: string, index: number) {
  console.log(index);
}

function log(this: any, ...args: any[]) {
  switch (args.length) {
    case 1:
      return logClass.apply(this, args); // 需要将 --strictBindCallApply 关掉
    case 2:
      return logProperty.apply(this, args);
    case 3:
      if (typeof args[2] === 'number') {
        return logParameter.apply(this, args);
      }
      return logMethod.apply(this, args);
    default:
      throw new Error('Decorators are not valid here!')
  }
}

@log
class Person { 
  @log
  public name: string;
  constructor(name : string) { 
    this.name = name;
  }
  @log
  public greet(@log message : string) : string { 
    return `${this.name} say: ${message}`;
  }
}
```

## 装饰器顺序

可以同时应用多个装饰器，装饰器可以写在一行或多行上，执行的步骤是：
1. 从上到下依次对装饰器表达式求值
2. 求值的结果会被当做函数，从下到上一次调用

```typescript
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```

在类中的各种装饰器调用顺序：

1. 参数装饰器、方法装饰器、访问符装饰器、属性装饰器应用到每个实例成员
2. 参数装饰器、方法装饰器、访问符装饰器、属性装饰器应用到每个静态成员
3. 参数装饰器应用到静态函数
4. 类装饰器应用到类

# Reflect Metadata

属于 ES7 的一个提案，主要作用是在声明的时候读取和添加元数据，不再需要手动在属性上添加元数据，目前需要先引入 npm 包：

```bash
npm i reflect-metadata --save
```

而且需要在 `tsconfig.json` 中配置 `emitDecoratorMetadata`

```typescript
@Reflect.metadata('name', 'A')
class A {
  @Reflect.metadata('hello', 'world')
  public hello(): string {
    return 'hello world'
  }
}

Reflect.getMetadata('name', A);
Reflect.getMetadata('hello', new A());
```

## 常用方法

### 设置/获取元数据

可以使用 `metadata` API，利用装饰器给目标添加元数据

```typescript
function metadata(
  metadataKey: any,
  metadataValue: any
): {
  (target: Function): void;
  (target: Object, propertyKey: string | symbol): void;
}
```

也可以使用 `defineMetadata` 添加元数据

```typescript
Reflect.defineMetadata(metadataKey, metadataValue, target);
Reflect.defineMetadata(metadataKey, metadataValue, target, propertyKey);
```

可以这样使用他：

```typescript
@Reflect.metadata('name', 'xiaomimg')
class Person {
  @Reflect.metadata('time', '2019/10/10')
  public say(): string {
    return 'hello';
  }
}

console.log(Reflect.getMetadata('name', Person)); // xiaomimg
console.log(Reflect.getMetadata('time', new Person, 'say')); // 2019/10/10
```

注意取出 `say` 上的元数据时需要先实例化，因为元数据是被添加在实例方法上的，要想不实例化的话，则需要添加在静态方法上

### 内置元数据

可以获取一些 TypeScript 本身内置的一些元数据

```typescript
const type = Reflect.getMetadata('design:type', new Person, 'say');
const typeParam = Reflect.getMetadata('design:paramtypes', new Person, 'say');
const typeReturn = Reflect.getMetadata('design:returntype', new Person, 'say')
```

# 明确赋值断言

`--strictPropertyInitialization` 可以保证变量声明和实例属性都会有初始值，但有时候我们需要提醒编译器 ”这里不需要一个初始值“ 或者 ”这里稍后将会有值，你别管“，这是加上 `!` 即可

# is 关键字

作用是为了判断类型，看一个栗子：

```typescript
function isString(test: any): test is string {
  return typeof test === 'string';
}

function example(foo: number | string) {
  if (isString(foo)) {
    console.log(foo.length);
  }
}

example('hello world')
```

如果将 `test is string` 改成 `boolean`，就会报错：

```text
Property 'length' does not exist on type 'string | number'.
  Property 'length' does not exist on type 'number'.(2339)
```

可以看到 `is` 将类型的范围给缩小了

# 可调用类型注解

想让一个 interface 被注解为可执行的、可实例化的：

```typescript
interface ToString {
  
}

declare const somethingToString: ToString;

somethingToString();

// This expression is not callable.
//    Type 'ToString' has no call signatures.
```

可以这样修改：

```typescript
interface ToString1 {
  (): string
}

declare const somethingToString1: ToString1;

somethingToString1();

// OR

interface ToString2 {
  new (): string
}

declare const somethingToString2: ToString2;

new somethingToString2();
```
