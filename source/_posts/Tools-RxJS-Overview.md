---
title: RxJS Overview
date: 2021-05-13 14:39:12
tags:
- 入门
categories:
- [工具, RxJS]
---

One Core Type: Observable.
Satellite Types: Observer, Schedulers, Subjects.
Operators: map, filter, reduce, every, etc.

<!-- more -->

# Observable

|  | SINGLE | MULTIPLE |
| --- | --- | --- |
| Pull | Function | Iterator |
| Push | Promise | Observable |

一个例子：

```typescript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  // 注意：当 subscribed 的时候，Observable push 1, 2, 3 的过程是同步执行的！ 
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');
observable.subscribe({
  next(x) { console.log('got value ' + x); },
  error(err) { console.error('something wrong occurred: ' + err); },
  complete() { console.log('done'); }
});
console.log('just after subscribe');
```

```text
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

## Pull 和 Push

在描述 **数据生产者(Producer)** 和 **数据消费者(Consumer)** 之间的沟通时，有 **Pull** 和 **Push** 两种不同的方案。

### Pull

消费者决定了何时从生产者那里拿数据。

比如每个 JS 函数就是一个 Pull System：函数是数据的生产者，调用函数的地方从函数那里 pull **一个** 返回值。
ES6 引入的 generator 函数和迭代器也是一种 Pull System：消费者调用 `iterator.next()`，从迭代器那里 pull **多个** 值。

### Push

生产者决定了何时发送数据给消费者。

比如 Promise(生产者) 给 回调函数(消费者) 传输数据，Promise 掌控了 push 数据的时机。
Observable(生产者) 给 Observer(消费者) 传输数据。

## Observable 与函数

{% cq %}
- function.call() 表示 *同步地给我一个值*
- observable.subscribe() 表示 *同步或异步地给我一堆值*
{% endcq %}

### Observable 与函数的相同点

#### 懒执行

如果不 call Function，那么函数就不会执行；
如果不 subscribe Observable，那么 Observable 中注册的回调函数也不会执行。

```typescript
function foo() {
  console.log('Hello');
  return 42;
}

const x = foo.call();
console.log(x);
const y = foo.call();
console.log(y);
```
```typescript
const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
});
 
foo.subscribe(x => {
  console.log(x);
});
foo.subscribe(y => {
  console.log(y);
});
```

运行上面两段代码的结果是一样的：

```text
"Hello"
42
"Hello"
42
```

#### 每个 subscribing 都是独立的

就像每次 Function calling 都是独立的一样，每次 Observable subscribing 也是独立的，他们之间的副作用互不干扰——这与 EventEmitter 不同。

#### Observable 可同步可异步

Observable 并不是都是异步的，如果没有引入其他异步数据源，他其实是同步的，这跟 Function 也是类似的。

```typescript
console.log('before');
console.log(foo.call());
console.log('after');
```
```typescript
console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

以上两段代码运行的结果也是一样的：

```text
"before"
"Hello"
42
"after"
```

### Observable 与函数的不同点

#### Observable 可以发出多个值

在函数中，我们不能这样写：

```typescript
function foo() {
  console.log('Hello');
  return 42;
  return 100; // dead code. will never happen
}
```

但是在 Observable 却可以这样写：

```typescript
const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100); // "return" another value
  subscriber.next(200); // "return" yet another
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```
```text
"before"
"Hello"
42
100
200
"after"
```

#### Observable 可以异步发出值

```typescript
const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100);
  subscriber.next(200);
  setTimeout(() => {
    subscriber.next(300); // happens asynchronously
  }, 1000);
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```
```text
"before"
"Hello"
42
100
200
"after"
300
```

## Creating Observable

Observable 构造函数只需要一个参数：subscribe 函数

```typescript
// 创建一个 Observable，他会给一个 subscriber 每秒钟都发出 "hi"
const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi')
  }, 1000);
});
```

## Subscribing Observable

```typescript
observable.subscribe(x => console.log(x));
```

当调用 `observable.subscribe` 的时候，subscribe 函数会用给定的 subscriber 运行，并且每次调用都是独立的。
与 `addEventListener` 这种 API 不一样，Observer 并没有注册为一个 listener，Observable 里面也没有维护 Observer 的列表。

## Executing Observable

在 `new Observable(function subscribe(subscriber) {...})` 中的代码表示一个 Observable execution，这个 execution 会随着时间通过 Next 发出多个值。但当发出 Error 或者 Complete 之后，就不会再发出值了。

## Disposing Observable Executions

当调用 `observable.subscribe` 之后，Observer 会跟一个新创建的 Observable execution 绑定。并且返回一个 `Subscription`：

```typescript
const subscription = observable.subscribe(x => console.log(x));
```

Subscription 就代表了正在执行的 execution。
- 可以通过 `subscription.unsubscribe()` 来取消正在执行的 execution。
- Complete 或 Error 也会取消 execution。

我们有时候需要自定义一个 `unsubscribe` 函数来在取消 execution 的时候释放资源。

```typescript
const observable = new Observable(function subscribe(subscriber) {
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  // 这个函数会在 unsubscribe 的时候执行，清除掉 interval
  return function unsubscribe() {
    clearInterval(intervalId);
  };
});
```
