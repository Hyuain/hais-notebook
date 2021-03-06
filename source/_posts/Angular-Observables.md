---
title: Angular：Observables
date: 2020-03-25 11:29:15
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, Angular]
---

- 提供了 publishers 和 subscribers 之间传递数据的方式
- 在 subscribe 的时候才会执行，而不是 define 的时候
- 可以传递多个任意类型的数据

<!-- more -->

# 基本使用

```typescript
// 创建一个 Observable 实例，他定义了一个 subscriber 函数
// 这个函数在 subscribe() 的时候才会执行
// 这个函数定义了如何获取数据、生成 values 和 messages
const locations = new Observable((observer) => {
  let watchId: number;
  if ('geolocation' in navigator) {
    watchId = navigator.geolocation.watchPosition((position: Position) => {
      observer.next(position);
    }, (error: PositionError) => {
      observer.error(error);
    })
  } else {
    observer.error('Geolocation not available');
  }

  return {
    unsubscribe() {
      navigator.geolocation.clearWatch(watchId);
    }
  }
})

// 调用 subscribe() 方法之后
// 并传入一个定义了 handler（next、error、complete，其中 next 是必须的）的对象（observer）
// 返回一个 Subscription 对象，其中有一个 unsubscribe() 方法
const locationSubscription = locations.subscribe({
  next(position) {
    console.log('Current Position: ', position);
  },
  error(message) {
    console.log('Error Getting Location: ', message);
  }
})

setTimeout(() => {
  locationSubscription.unsubscribe();
}, 10000)
```

## stream

observable 可以表示任意数据类型，我们将 observable 发布（publish）出来的数据称为流（stream）

## custom fromEvent function

```typescript
function fromEvent(target, eventName) {
  return new Observable((observer) => {
    const handler = (e) => observer.next(e);
    target.addEventListener(eventName, handler);  
    return () => {
      target.removeEventListener(eventName, handler);
    }
  })
}

const ESC_KEY = 27;
const nameInput = document.getElementById('name') as HTMLInputElement;
const subscription = fromEvent(nameInput, 'keydown')
  .subscribe((e: KeyboardEvent) => {
    if (e.keyCode === ESC_KEY) {
      nameInput.value = '';
    }
  });
```

## Multicasting

通常一个 observable 会为每一个调用了 subscribe() 的 observer 创建一次新的、独立的执行。当 observer 调用 subscribe() 的时候，observable 会连接上一个 event handler 并将值传递给 observer。

但是在有的时候，我们可能会想要每个 subscription 都拿到同样的值，这就叫 multicasting。下面是一个例子：

```typescript
function sequenceSubscriber(observer) {
  const seq = [1, 2, 3];
  let timeoutId;

  function doSequence(arr, idx) {
    timeoutId = setTimeout(() => {
      observer.next(arr[idx]);
      if (idx === arr.length - 1) {
        observer.complete();
      } else {
        doSequence(arr, ++idx);
      }
    }, 1000);
  }
 
  doSequence(seq, 0);

  return {
    unsubscribe() {
      clearTimeout(timeoutId);
    }
  }    
}

const sequence = new Observable(sequenceSubscriber);

sequence.subscribe({
  next(num) { console.log('1st subscribe' + num); },
  complete() { console.log('1st sequence finished.'); }
});

// 如果 subscribe 两次，那么就会产生两个不同的 stream

setTimeout(() => {
  sequence.subscribe({
    next(num) { console.log('2nd subscribe' + num); },
    complete() { console.log('2nd sequence finished.'); }
  });
}, 500);
```

输出如下：

```text
(at 1 second): 1st subscribe: 1
(at 1.5 seconds): 2nd subscribe: 1
(at 2 seconds): 1st subscribe: 2
(at 2.5 seconds): 2nd subscribe: 2
(at 3 seconds): 1st subscribe: 3
(at 3 seconds): 1st sequence finished
(at 3.5 seconds): 2nd subscribe: 3
(at 3.5 seconds): 2nd sequence finished
```

将其转换为 multicasting observer：

```typescript
function multicastSequenceSubscriber() {
  const seq = [1, 2, 3];
  const observers = [];
  // 只会产生一组值，被广播给每个 subscriber，因此只有一个 timerId
  let timeoutId;

  // 返回 subscriber function，当执行 subscribe() 的之后被调用
  return (observer) => {
    observers.push(observer);
    // 如果是第一个 subscription，开始这个 sequence
    if (observers.length === 1) {
      timeoutId = doSequence({
        next(val) {
          // 遍历所有的 observers 并通知所有的 subscriptions
          observers.forEach(obs => obs.next(val));
        },
        complete() {
          observers.slice(0).forEach(obs => obs.complete());
        }
      }, seq, 0)
    }
    
    return {
      unsubscribe() {
        observers.splice(observers.indexOf(observer), 1);
        if (observers.length === 0) {
          clearTimeout(timeoutId);
        }
      }
    }
  }
}

function doSequence(observer, arr, idx) {
  return setTimeout(() => {
    observer.next(arr[idx]);
    if (idx === arr.length - 1) {
      observer.complete()
    } else {
      doSequence(observer, arr, ++idx)
    }
  }, 1000)
}

const multicastSequence = new Observable(multicastSequenceSubscriber());

multicastSequence.subscribe({
  next(num) { console.log('1st subscribe: ' + num); },
  complete() { console.log('1st sequence finished.'); }
})

// 1.5s 后再次 subscribe，不会发送第一个值
setTimeout(() => {
  multicastSequence.subscribe({
    next(num) { console.log('2nd subscribe: ' + num); },
    complete() { console.log('2nd sequence finished.'); }
  });
}, 1500);
```

# RxJS

RxJS ( Reactive Extensions for JavaScript ) 是一个响应式编程（一种考虑异步数据流和数据变化）库

## Create Observable

RxJS 提供多种函数，可以通过多种方法从事件、Promise、计时器等创建 observable：

```typescript
// from
const data = from(fetch('/api/endpoint'));
data.subscribe({
  next(response) { console.log(response); },
  error(err) { console.error('Error: ' + err); },
  complete() { console.log('Completed'); }
})

// interval
const secondsCounter = interval(1000);
secondsCounter.subscribe(n => console.log(`It's been ${n} seconds since subscribing!`))

// fromEvent
const el = document.getElementById('my-element');
const mouseMoves = fromEvent(el, 'mousemove');
const subscription = mouseMoves.subscribe((event: MouseEvent) => {
  console.log(`Coords: ${event.clientX} X ${event.clientY}`);
  if (event.clientX < 40 && event.clientY < 40) {
    subscription.unsubscribe();
  }
})

// ajax
import { ajax } from 'rxjs/ajax';
const apiData = ajax('/api/data');
apiData.subscribe(res => console.log(res.status, res.response));
```

## Operators

操作符执行操作，转换原来的 observable 值，并且返回一个包含转换后的值的 observable：

```typescript
const nums = of(1, 2, 3);

const squareValues = map((val: number) => val * val);
const squareNums = squareValues(nums);

squareNums.subscribe(console.log)
```

可以使用 `pipe` 将操作符连接在一起，我们将一组被应用在 observable 上的操作符称为一个 recipe，需要调用 `subscribe()` 来生成这个 recipe 操作之后的结果。

```typescript
const nums = of(1, 2, 3, 4, 5);

const squareOddVals = pipe(
  filter((n: number) => n % 2 !== 0),
  map(n => n * n)
)

const squareOdd = squareOddVals(nums);

// 也可以这样写：
// const squareOdd = nums.pipe(
//   filter((n: number) => n % 2 !== 0),
//   map(n => n * n)
// );

squareOdd.subscribe(console.log)
```

## Error Handling

除了 subscription 中的 error handler 之外，还可以使用 `catchError` 操作符来处理错误，通过这个操作符可以提供一个备用的解决方案，并且不中断运行。

```typescript
const apiData = ajax('/api/data').pipe(
  map(res => {
    if (!res.response) {
      throw new Error('Value expected!');
    }
    return res.response;
  }),
  catchError(err => of([]))
);

apiData.subscribe({
  next(x) { console.log('data: ', x); },
  error(err) { console.log('errors already caught and this will not run'); }
})
```

`catchError` 可以让错误快速恢复，搭配 `retry` 还可以进行多次重试：

```typescript
const apiData = ajax('/api/data').pipe(
  retry(3),
  map(res => {
    if (!res.response) {
      throw new Error('Value expected!');
    }
    return res.response;
  }),
  catchError(err => of([]))
);
``` 

# Angular 中的 Observables

## 组件间传值

Angular 提供了 `EventEmitter` API，通过 `@Output()` 就可以发布数据，`EventEmitter` 就是基于 RxJS Subject 的。
当 `emit()` 的时候，他会将值传递给任意订阅了的 observer 的 `next()` 方法。下面是一个例子：

```html
<!-- 在父组件监听自定义的 open 和 close 事件 -->
<zippy (open)="onOpen($event)" (close)="onClose($event)"></zippy>
```

```typescript
// 在子组件 emit 这两个自定义事件，并传递值
@Component({
  selector: 'zippy',
  template: `
  <div class="zippy">
    <div (click)="toggle()">Toggle</div>
    <div [hidden]="!visible">
      <ng-content></ng-content>
    </div>
  </div>`})
export class ZippyComponent {
  visible = true;
  @Output() open = new EventEmmiter<any>();
  @Output() close = new EventEmmiter<any>();

  toggle() {
    this.visible = !this.visible;
    if (this.visible) {
      this.open.emit(null);
    } else {
      this.close.emit(null);
    }
  }
}
```

## HTTP

Angular 的 `HttpClient` API 也将会返回 observables，这有一些优点：

- Observables 不会修改从服务端获取的数据
- 可以通过 `unsubscribe()` 方法取消请求
- 可以配制请求

## Async pipe

AsyncPipe 会 subscribe 一个 observable 或 promise，并且返回他发出的最新值

## Router

`Router.events` 也是以 observables 的形式提供事件的：

```typescript
import { Router, NavigationStart } from '@angular/router';
import { filter } from 'rxjs/operators';

@Component({
  // ...
})
export class RoutableComponent implements OnInit {
  navStart: Observable<NavigationStart>;
  
  constructor(private router: Router) {
    this.navStart = router.events.pipe(
      filter(event => event instanceof NavigationStart)
    ) as Observable<NavigationStart>
  }

  ngOnInit() {
    this.navStart.subscribe(event => console.log('Navigation Started!'));
  }
}
```

# 区分 Observables 和其他技术

## Observables 和 Promises

- Observables 是 declarative，当调用 `subscribe()` 的时候才会执行，而 Promise 在定义后会马上执行
- Observables 提供多个值，而 Promises 值提供一个值
- Observables 有链式调用和 subscribe()

### 创建与订阅

- Observables 在 consumer subscribe 的时候才会执行，并且每个 subscription 都拥有自己独立的计算
- Promises 在创建的时候执行，并且在创建的时候就已经开始计算结果了，每个 then 分享同样的结果，并且不能中断或者重启

### 链式调用

- Observables 将转换数据与最终获取（订阅）的过程分开了，只有订阅（subscription）才会让 subscriber 函数开始计算结果
- Promises 并不会区分中间与最后的 then

### 取消

- Observables 可以在 subscriber 函数中返回 unsubscribe 方法，并可使用 `subscription.unsubscribe()` 进行调用
- Promises 则无法取消

### 错误处理

- Observables 会将错误传递给 subscriber 的 error handler（在调用 `subscribe()` 的时候定义）
- Promises 将错误传递给子 Promises

## Observables 与 EventHandlers

- 都定义了通知处理方法（notification handlers），并且通过他们来随着时间变化的处理多个值
- subscribe 一个 observables 相当于添加一个事件监听，不同在于 observables 可以在传递给 handler 之前对数据进行转换和处理
- 可以在比如 HTTP 请求之类的异步操作中使用 Observables 来处理事件，这样可以增加上下文的紧密度

```typescript
/*
   都需要定义 handlers
*/
function handler(e) {
  console.log('Clicked', e);
}

/*
   Events API
*/
// 创建并开始监听
buttonEl.addEventListener('click', handler);
// 取消监听
buttonEl.removeEventListener('click', handler);
/*
   Observable
*/
// 创建
const clicks$ = fromEvent(buttonEl, 'click');
// 开始监听
const subscription = clicks$.subscribe(handler)
// 取消监听
subscription.unsubscribe()
```
