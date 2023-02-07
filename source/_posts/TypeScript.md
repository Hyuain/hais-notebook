---
title: TypeScript
date: 2020-03-21 17:25:49
categories:
  - [前端]
---

TypeScript 三个好处： 1. 自动提示更智能；2. 不能随便写 .xxx()；3. 错误前移到编译时 

<!-- more -->

# Typescript 中的类型

## 原始类型

TS 中的原始类型包括 `boolean` `number` `string` `void` `undefined` `null` `symbol` `bigint`，与 JS 不同的是多了 `void` 和 `bigint` 这两个：

### void

表示没有任何类型，相当于 `undfined | null`，通常用在一个函数没有返回值时（`undefiend`）

```typescript
const a: void = undefined;
```

### bigint

主要是为了解决大数超过精度范围的问题的，需要注意的是 `bigint` 与 `number` 是两种类型

```typescript
const max: bigint = BigInt(Number.MAX_SAFE_INTEGER);

const max1: bigint = max + 1n;
const max2: bigint = max + 2n;

max1 !== max2 // true
```

## 其他类型

- **顶级类型**：`any` `unknown`
  *计算机类型系统理论中的顶级类型（Top Type）通常是指某种包含了所有可能类型的类型，其他所有的类型都是其子类型

- **底部类型**：`never`
  *与顶级类型相反，底部类型（Bottom Type）又称为零类型（Zero）或空类型（Empty），表示什么都没有，他是任何类型的子类型，甚至连 `undefiend` 或者 `null` 都不能赋值给 `never`

- **非原始类型**：`object` `array` `tuple` 等

### any

有些时候，我们在编程阶段还不知道某个变量的类型，比如来自用户的输入或者第三方代码库，我们用 `any` 可以让他通过类型检查

{% note warning %}
但是注意，尽量不用 `any`，因为会造成蝴蝶效应，可能导致很严重的问题
{% endnote %}

```typescript
let notSure: any = 4;
notSure = 'hello';
```

### unknown

`unknow` 是 TypeScript 3.0 引入的比 `any` 更加安全的类型，区别是有时候需要一些必要的检查：

```typescript
let value: any;

value = true;
value = 1;
value = 'hello';
value = Symbol('type');
value = {};
value = [];
```

`unknown` 也可以是任何类型：

```typescript
let value: unknown;

value = true;
value = 1;
value = 'hello';
value = Symbol('type');
value = {};
value = [];
```

区别在于：

```typescript
let value: any;

value.foo.bar;
value();
new value();
value[0][1];
```

而 `unknow` 则不能

```typescript
let value: unknown;

value.foo.bar; // ERROR
value();       // ERROR
new value();   // ERROR
value[0][1];   // ERROR
```

你得通过某种方式确认 `value` 的类型：

```typescript
function getValue(value: unknown): string {
  if (value instanceof Date) {
    return value.toISOString()
  }
  return String(value)
}
```

### never

`never` 是任何类型的子类型，可以赋值给任何类型，然而没有别的类型可以赋值给 `never`，下面是一些例子：

```typescript
// 抛出异常的函数不会有返回值
function error(message: string): never {
  throw new Error(message)
}

// 空数组，而且永远是空的
const empty: never[] = []
```

### array

有两种方式可以定义一个数组：

- 使用泛型：

```typescript
const list: Array<number> = [1, 2, 3];
```

- 在元素后面加上 `[]`

```typescript
const list: number[] = [1, 2, 3];
```

### tuple

元祖与数组的不同是，元素的各元素类型不必相同：

```typescript
let x: [string, number];
x = ['hello', 10, false]; // ERROR
x = ['hello'];            // ERROR
x = [10, 'hello'];        // ERROR
x = ['hello', 10];        // OK
```

注意元祖元素的顺序不同也会报错，类型检查比数组严格；并且元祖还有越界问题：

```typescript
const tuple: [string, number] = ['a', 1];
tuple.push(2); // OK
console.log(tuple); // 元素正常打印
console.log(tuple[2]); // Tuple type '[string, number]' of length '2' has no element at index '2'
```

### object

事实上，普通对象、枚举、数组、元素都是 `object` 类型

```typescript
enum Direction {
  Center = 1
}

let value: object;

value = Direction;
value = [1];
value = [1, 'hello'];
value = {};
```

## 枚举类型

通常在某个变量只有几种可能的取值时，可以使用枚举类型

### 数字枚举

若没有赋值，默认就是从 0 开始的数字枚举：

```typescript
enum Direction {
  Up,
  Down,
  Left,
  Right
}

Direction.Up === 0;   // true
Direction.Down === 1; // true
```

若给第一个赋值为数字，则后面则会累加：

```typescript
enum Direction {
  Up = 10,
  Down,
  Left,
  Right
}

Direction.Up === 10;   // true
Direction.Down === 11; // true
```

### 字符串枚举

枚举类型的值也可以是字符串类型：

```typescript
enum Direction {
    Up = 'Up',
    Down = 'Down',
    Left = 'Left',
    Right = 'Right'
}
```

### 异构枚举

但是通常不这样写

```typescript
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

### 反向映射

数字枚举可以进行反向映射，因为他会被编译成诸如 `Direction[Direction["Up"] = 0] = "Up"` 的语句，*而字符串枚举则不具备这样的特性*，并且也不推荐这样做（`Potentially invalid target of indexed property access`）

```typescript
enum Direction {
    Up,
    Down,
    Left,
    Right
}

console.log(Direction[0]); // Up
```

### 常量枚举

我们可以将枚举使用 `const` 声明为常量：

```typescript
const enum Direction {
  Up = 'Up',
  Down = 'Down'
}

const a = Direction.Up
```

这时将会被编译为 `const a = 'Up'`，而不是之前的 `const a = Direction.Up`——这可以提升性能，除非添加编译选项 `--preserveConstEnums` 使他保留对象 `Direction`

### 联合枚举

```typescript
enum Direction {
  Up,
  Down
}

declare let a: Direction; // 可以看成声明了联合类型 Direction.Up | Direction.Down | Direction.Left | Direction.Right

enum Animal {
  Dog,
  Cat
}

a = Direction.Up; // OK
a = Animal.Dog;   // Assigned expression type Animal.Dog is not assignable to type Direction
```

### 枚举合并

分开声明的枚举会自动合并，而不会发生冲突

```typescript
enum Direction {
  Up = 'Up',
  Down = 'Down'
}
enum Direction {
  Center = 1
}
```

# interface

一个简单的栗子，包括只读属性、可选属性和函数：

```typescript
interface User {
  name: string
  age?: number
  readonly isMale: boolean
  say: (words: string) => string
}
```

## 属性检查

有时候会出现这样的情况：

```typescript
interface Config {
  width?: number
}

function calc(config: Config): number {
  let square = 100;
  if (config.width) {
    square = config.width * config.width
  }
  return square
}

let mySquare = calc({x: 5}) // Argument type {x: number} is not assignable to parameter type Config
```

解决办法：

- 使用类型断言

```typescript
let mySquare = calc({x: 5} as Config)
```

- 添加字符串索引签名，这样 `Config` 就可以拥有任意数量的属性了

```typescript
interface Config {
  width?: number
  [propName: string]: any
}
```

- 转化为 `any`

## 可索引属性

```typescript
interface Config {
  [propName: string]: string
}
```

## 继承接口

```typescript
interface VIPUser extends User, SuperUser {
  broadcast: () => void
}
```

# class

## 抽象类

`abstract` 关键字可以用于定义一个抽象类或在抽象类中定义抽象方法，抽象类是专门用来派生的，一般不会被实例化

```typescript
abstract class Animal {
  abstract makeSound(): void;
  move(): void {
    console.log('roaming the earth...')
  }
}

// 不能直接使用 new Animal() 创建实例

class Cat extends Animal {
  makeSound() {
    console.log('miaou~')
  }
}

const cat = new Cat();

cat.makeSound(); // 'miaou~'
cat.move(); // 'roaming the earth...'
```

## 访问限定符

包括 public、private、protected

### public

TS 中成员默认为 public，可以被外部访问：

```typescript
class Car {
  public run() {
    console.log('start...');
  }
}

const car = new Car();
car.run(); // 'start...'
```

### private

private 成员只能被类的内部访问

### protected

protected 成员可以被类的内部以及子类访问

```typescript
class Car {
  protected run() {
    console.log('start...');
  }
}

class GTR extends Car {
  init() {
    this.run();
  }
}

const car = new Car();
const gtr = new GTR();

car.run(); // Protected member is not accessible
gtr.init(); // 'start...'
gtr.run(); // Protected member is not accessible
```

## class 作为 interface

在 React 中比较常用

```typescript
// props 的类型
class Props {
  public children: Array<React.ReactElment<any>> | React.ReactElment<any> | never[] = [];
  public speed: number = 500;
  public height: number = 160;
  // ...
}

export default class Carousel extends React.Component<Props, State> {
  public static defaultProps = new Props()
}
```

# function

## 定义函数类型

很多时候函数的类型并不需要刻意去定义，TypeScript 会自己进行类型推断：

```typescript
const add = (a: number, b = 10, c?: number) => a + b + (c ? c : 0);
// 当然也可以显式定义
const mul: (a: number, b: number) => number = (a: number, b: number) => a * b;
```

当然也可以使用 `...` 来表示剩余参数

```typescript
const add = (a: number, ...rest: number[]) => rest.reduce(((a, b) => a + b), a)
```

## overload

```typescript
interface Direction {
  top: number,
  bottom?: number,
  left?: number,
  right?: number
}
function assigned(all: number): Direction
function assigned(topAndBottom: number, leftAndRight: number): Direction
function assigned(top: number, right: number, bottom: number, left: number): Direction

function assigned (a: number, b?: number, c?: number, d?: number) {
  if (b === undefined && c === undefined && d === undefined) {
    b = c = d = a;
  } else if (c === undefined && d === undefined) {
    c = a;
    d = b;
  }
  return {
    top: a,
    right: b,
    bottom: c,
    left: d
  }
}

assigned(1);
assigned(1,2);
assigned(1,2,3); // Argument types do not match parameters 
assigned(1,2,3,4);
```

接受不同个数的参数，实现函数的重载，但是需要对函数做三次类型声明，否则可能会在传入 3 个参数（代码事实上不允许传入 3 个参数）的时候不报错

## generics

有些时候，我们在静态编写的时候并不确定传入的参数到底是什么类型，只有在运行时传入参数之后才能确定。那么就需要一个变量，用于表示类型，而这个类型变量，则称为 **泛型（generics）**。

```typescript
// 在函数名称后面声明泛型变量 <T>
function returnItem<T>(para: T): T {
  return para
}
```

可以同时有多个泛型变量：

```typescript
function swap<T, U>(tuple: [T, U]) {
  return [tuple[1], tuple[0]]
}

swap([7, 'seven'])
```

## 泛型变量

有时候我们可能会写出这样的代码：

```typescript
function getArrayLength<T>(arg: T): T {
  console.log(arg.length); // 类型 T 上不存在 length
  return arg
}
```

编译器并不知道 `T` 上有 `length` 这个属性，我们可以写成 `Array<T>`

```typescript
function getArrayLength<T>(arg: Array<T>) {
  console.log((arg as Array<any>).length);
  return arg
}
```

## 泛型接口

```typescript
interface ReturnItemFn<T> {
  (para: T): T
}
const returnItem: ReturnItemFn<number> = para => para
```

## 泛型类

泛型可以作用域类本身，也可以作用于成员函数

```typescript
class Stack<T> {
  private arr: T[] = [];
  public push(item: T) {
    this.arr.push(item)
  }
  public pop() {
    this.arr.pop()
  }
}
```

## 泛型约束

```typescript
type Params = number | string

class Stack<T extends Params> {
  private arr: T[] = [];
  public push(item: T) {
    this.arr.push(item)
  }
  public pop() {
    this.arr.pop()
  }
}
```

## 泛型约束与索引类型

```typescript
function getValue(obj: object, key: string) {
  return obj[key] // ERROR
}
```

需要这样修改：

```typescript
function getValue<T extends object, U extends keyof T>(obj: T, key: U) {
  return obj[key] // OK
}
```

## 泛型与 new

我们需要这样声明一个泛型为构造函数：

```typescript
function factory<T>(Constructor: {new(): T}) {
  return new Constructor()
}
```

# 类型断言与类型守卫

## 类型断言

```typescript
interface Person {
  name: string
  age: number
}

const person = {} as Person;

person.name = 'xiaoming'; // OK
person.age = 20;          // OK
```

## 双重断言

```typescript
interface Person {
  name: string
  age: number
}

const person = 'xiaoming' as any as Person;  // OK
```

## 类型守卫

使用类型守卫可以缩小类型的范围

### instanceof

```typescript
class Person {
  name = 'xiaoming';
  age = 20;
}

class Animal {
  name = 'petty';
  color = 'pink';
}

function getSometing(arg: Person | Animal) {
  // 类型细化为 Person
  if (arg instanceof Person) {
    console.log(arg.color); // Error，因为arg被细化为Person，而Person上不存在 color属性
    console.log(arg.age); // OK
  }
  // 类型细化为 Person
  if (arg instanceof Animal) {
    console.log(arg.age); // Error，因为arg被细化为Animal，而Animal上不存在 age 属性
    console.log(arg.color); // OK
  }
}
```

### in

```typescript
class Person {
  name = 'xiaoming';
  age = 20;
}

class Animal {
  name = 'petty';
  color = 'pink';
}

function getSometing(arg: Person | Animal) {
  if ('age' in arg) {
    console.log(arg.color); // Error，因为arg被细化为Person，而Person上不存在 color属性
    console.log(arg.age); // OK
  }
  if ('color' in arg) {
    console.log(arg.age); // Error，因为arg被细化为Animal，而Animal上不存在 age 属性
    console.log(arg.color); // OK
  }
}
```

### 字面量类型守卫

```typescript
type Foo = {
  kind: 'foo'; // 字面量类型
  foo: number;
};

type Bar = {
  kind: 'bar'; // 字面量类型
  bar: number;
};

function doStuff(arg: Foo | Bar) {
  if (arg.kind === 'foo') {
    console.log(arg.foo); // ok
    console.log(arg.bar); // Error
  } else {
    console.log(arg.foo); // Error
    console.log(arg.bar); // ok
  }
}
```

# 类型兼容性

## 结构类型

结构类型是一种只使用其成员来表示类型的方式，一个栗子：

```typescript
class Person {
  constructor(public weight: number, public name: string, public born: string) {}
}
interface Dog {
  name: string
  weight: number
}

let x: Dog;

x = new Person(120, 'xx', '22-22-22') // OK，多的可以赋值给少的
```

这时我们说 `Dog` 兼容 `Person`， 因为 `Dog` 的属性 `Person` 都有，反之则不然。

## 函数类型

首先查看参数列表（的类型）：

```typescript
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Assigned expression type (b: number, s: string) => 0 is not assignable to type (a: number) => 0
```

少的可以赋值给多的

## 枚举类型的兼容性

枚举与数字类型相互兼容

```typescript
enum Status {
  Ready,
  Waiting
}

let status = Status.Ready;
let num = 0;

status = num;
num = status;
```

## 类的类型兼容性

只有实例成员和方法会被比较，构造函数和静态成员则不会被检查：

```typescript
class Animal {
  feet: number;
  constructor(name: string, numFeet: number) {}
}

class Size {
  feet: number;
  constructor(meters: number) {}
}

let a: Animal;
let s: Size;

a = s; // OK
s = a; // OK
```

而 private 和 protected 必须来自相同的类

# Vue 的 TS 组件

```typescript
import Vue from 'source/_posts/Others-QA-Vue';
import {Component, Prop} from 'vue-property-decorator';

// 这是一个装饰器，告诉 Vue 这是一个组件，Vue 会自动把下面的东西处理成 data、method 等
@Component({
  // components: {Child,...}
})
export default class MyComponent extends Vue {
  num = 0; // 这就是 TS 组件的 data

  @Prop(Number) propA: number | undefined; // 这是 vue-property-decorator 带来的 props 写法
  // @Prop 是装饰器，告诉 Vue，后面的东西不是 data，是 prop
  // 左边的 Number 是告诉 Vue，propsA 是 Number，是**运行时**的类型检查
  // 右边的 number 是告诉 TS propsA 的类型，是**编译时**的类型检查，比如说你没办法写 this.propA.xxx，会无法编译成功

  add(num: number) { // 这就是 TS 组件的 methods
    this.num = num ++;
  }

  mounted() {
    console.log(this.propA);
  }
}
```

## 编译时与运行时

TypeScript --编译--> JavaScript --运行--> 浏览器

编译时：编译错误 -> 无法得到 JS，会在编译的终端 Error
运行时：运行错误 -> 控制台 Error（浏览器）

# TypeScript 的本质

{% cq %}
JS: 类型
{% endcq %}

TSC（TypeScriptCompiler）会利用类型来检查 JS 代码：

- 若检查出错误，就会编译报错，但还是会编译成 JS
- 若没有检查出错误，就会删掉类型，然后编译成 JS（TSC 或者 Babel 都可以编译，有一点点区别）

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

# 一些问题

## 强制指定类型

```typescript
inputNumber(event: MouseEvent | TouchEvent) {
  const button = (event.target as HTMLButtonElement); // 因为有的元素的 textContent 可能为空（比如图片），所以我们需要强制指定为 Button 元素
  console.log(button.textContent);
}

// 也可以给返回值强制指定类型
function Func() {
  return JSON.parse(localStorage.getItem('records') || '[]') as RecordItem[];
}
```

## 配合 JS 使用

```typescript
const xxx require('...')
```

## 全局声明 `type`

在 `src` 中创建 `custom.d.ts`（也可以叫 `xxx.d.ts`），在里面声明 `type`



## void

一般来讲 `void` 就是 `undefined` 或者 `null`，但在 `sctricNullChecks` 关闭时才可以取 `null`。
