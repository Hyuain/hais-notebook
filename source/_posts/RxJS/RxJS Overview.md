---
title: RxJS
date: 2021-05-13 14:39:12
categories:
  - - 前端
tags:
  - F1
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
function.call() 表示 *同步地给我一个值*
observable.subscribe() 表示 *同步或异步地给我一堆值*
{% endcq %}

### Observable 与函数的相同点

#### 懒执行

如果不 call Function，那么函数就不会执行；如果不 subscribe Observable，那么 Observable 中注册的回调函数也不会执行。

**注意这一点与 **

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

在 `new Observable(function subscribe(subscriber) {...})` 中的代码表示一个 Observable Execution。

Observable Execution 可以发出三种类型的值：

- "Next" 通知：会伴随一个 Number、String、Object 等类型的值
- "Error" 通知：会伴随一个 JS Error 或者异常
- "Complete" 通知：不会伴随任何值

通常来说，Execution 会随着时间通过 Next 通知发出多个值。但当发出 Error 或者 Complete 通知之后，就不会再发出值了。

## Disposing Executions

当调用 `observable.subscribe` 之后，Observer 会跟一个新创建的 Observable Execution 绑定。并且返回一个 `Subscription`：

```typescript
const subscription = observable.subscribe(x => console.log(x));
```

Subscription 就代表了正在执行的 execution。

- 可以通过 `subscription.unsubscribe()` 来取消正在执行的 Execution。
- Complete 或 Error 也会取消 execution。

我们有时候需要自定义一个 `unsubscribe` 函数来在取消 Execution 的时候释放资源。

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

# Observer

Observer 是 Observable 发出数据的消费者，其实就是一个对象，对象包含了三个分别用于处理 Observable 三种通知类型的回调函数。

```typescript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};

observable.subscribe(observer)

// 当然也可以直接将回调函数作为参数传进去，他会自动帮我们创建 Observer 对象
observable.subscribe(() => console.log('Observer got a next value: ' + x))
```

# Operators

Operator 就是函数。有两种类型的 Operator：

- Pipeable Operator：不会改变原来的 Observable 实例，他接受一个 Observable 作为参数，然后再返回一个 Observable
- Creation Operator：会创建一个新的 Observable

## 高阶 Observable

可以这样构造一个高阶 Observable，用 map 返回一个 Observable：

```typescript
const clicks = fromEvent(document, 'click');
const higherOrder = clicks.pipe(map(ev => of(ev)));
```

这时候就需要用例如 concatAll 等操作符将其拍平。

# Subscription

Subscription 通常是一个 Observable 的 Execution，主要是用来调用释放资源 API `unsubscribe`。

```typescript
const subscription = interval(400).subscribe(x => console.log('first: ' + x));
const childSubscription = interval(300).subscribe(x => console.log('second: ' + x));

subscription.add(childSubscription);

setTimeout(() => {
  // 同时 unsubscribe subcription 和 childSubscription
  subscription.unsubscribe();
}, 1000);
```

# Subject

Subject 是一种特殊的 Observable，他可以 **向多个 Observer 以多播的形式发出值**——而普通的 Observable 是单播，每个订阅的 Observer 对这个 Observable 都有自己独立的 Execution。

- **每个 Subject 都是 Observable**。你可以 `subscribe` 一个 Subject。对于 Subject 来说，`subscribe` 并不会产生一个新的 Execution，他会将这个 Observer 注册到一个 ObserverList，就像 `addListner` 一样
- **每个 Subject 都是 Observer**。他有 `next()` `error()` `complete()` 方法。只需要调用 `next()`，就会广播给所有已经注册了的 Observer。

```typescript
const subject = new Subject<number>();

// 每个 Subject 都是 Observable：可以 subscribe 他
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
```
```typescript
// 每个 Subject 都是 Observer：调用他的 next() 方法，就会广播给所有已经注册的 Observer
subject.next(1);
subject.next(2);

// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
```
```typescript
// 每个 Subject 都是 Observer：可以把他传给 subscribe 函数
from([1, 2, 3]).subscribe(subject);
 
// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

## 多播 Observable

注意！在 RxJS 7 已经将多播 API 进行了调整！[文档](https://rxjs.dev/guide/subject#multicasted-observables)中的 `multicast` 操作符将不再适用。详情请查看 [这篇文档](https://rxjs.dev/deprecations/multicasting)。

### connectable

可以通过 connectable 创建一个可以多播的 Observable：


## BehaviorSubject

他会保存发给消费者的最后一个值，并且当一个新的 Observer subscribe 的时候，他会从 `BehaviorSubject` 那里立即接收到这个值。

> BehaviorSubject 非常适合用于表示 “随着时间发出值”。比如表示 “生日” 事件的流是 Subject，但表示 “年龄” 的流是 BehaviourSubject。

```typescript
// 这里 0 表示初始值
const subject = new BehaviorSubject<number>(0);

subject.subscribe(res => {
  console.log('subscribe1', res);
});

subject.next(1);

setTimeout(() => {
  console.log('before');
  subject.subscribe(res => {
    console.log('subscribe2', res);
  });
  console.log('after');
}, 1000);

// Logs:
// subscribe1 0
// next!
// subscribe1 1
// before
// subscribe2 1
// after
```

## ReplaySubject

他跟 `BehaviorSubject` 很像，不同的是 `BehaviourSubject` 只会保存最后一个值，而 `ReplaySubject` 会保存很多个值。

```typescript
const subject = new ReplaySubject(3, 500); // 会缓存 3 个最新的值，最久保存 500ms
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(5);
 
// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```

## AsyncSubject

当 Observable Execution Complete 的时候才会发出值，并且只会发出最后一个值——这跟 `last()` 操作符差不多。

```typescript
const subject = new AsyncSubject();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});

subject.next(5);
subject.complete();

// Logs:
// observerA: 5
// observerB: 5
```

## Void Subject

简单来说，Subject 可以发出空值：

```typescript
const subject = new Subject<void>();
setTimeout(() => subject.next(), 1000);
```

# Scheduler

Scheduler 可以控制 subscription 开始和发出值的时间：

- **Scheduler 是一个数据结构**：他知道如何存储数据，并可以基于优先级来管理任务队列
- **Scheduler 是一个执行上下文**：他代表了任务在何时、何地执行（比如立即、或在某个回调中）
- **Scheduler 有一个虚拟时钟**：他提供了 `noew()` 函数来表示时间，由 Scheduler 管理的任务都依附于这个时间。

```typescript
import { Observable, asyncScheduler } from 'rxjs';
import { observeOn } from 'rxjs/operators';

// 这里的 proxyObserver 实际上是由 observeOn(asyncScheduler) 根据 finalObserver 创建的
const observable = new Observable((proxyObserver) => {
  proxyObserver.next(1);
  proxyObserver.next(2);
  proxyObserver.next(3);
  proxyObserver.complete();
}).pipe(
  observeOn(asyncScheduler)
);

const finalObserver = {
  next(x) {
    console.log('got value ' + x)
  },
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  complete() {
     console.log('done');
  }
};

console.log('just before subscribe');
observable.subscribe(finalObserver);
console.log('just after subscribe');

// LOGS:
// just before subscribe
// just after subscribe
// got value 1
// got value 2
// got value 3
// done
```

一个 `observeOn(asyncScheduler)` 创建的 `proxyObserver` 可以近似理解为：

```typescript
const proxyObserver = {
  next(val) {
    asyncSheduler.shedule(
      (x) => finalObserserver.next(x),
      0, // 延迟
      val, // 会传给上面的 x
    )
  }
}
```

异步的 Scheduler 是通过 `setTimeout` 或 `setInterval` 操作，因此就算延迟是 0，也是异步操作。

## Scheduler 的类型

| SCHEDULER | 目的 |
| --- | --- |
| null | 通知都是同步地、递归地发出的，用于正常时间的操作和尾递归 |
| queueScheduler | 以队列的形式在当前事件帧安排，用于迭代操作 |
| asapScheduler | 在同 promise 一样的微任务队列中安排，用于异步会话 |
| asyncScheduler | 使用 `setInterval`，用于基于时间的操作 |
| animationFrameScheduler | 任务会在浏览器下一次重绘（repaint）之前进行，用于创建流畅的浏览器动画 |

## 使用 Scheduler

以后用到再翻译，先看[文档](https://rxjs.dev/guide/scheduler)
