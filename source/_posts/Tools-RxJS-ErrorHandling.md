---
title: 翻译：RxJS 中的错误处理
date: 2021-03-03 22:47:21
tags:
- 入门
  categories:
- [工具, Webpack]
---

本文是 [RxJS Error Handling](https://blog.angular-university.io/rxjs-error-handling/) 的翻译。
主要介绍了 catchError、throwError、Finalize、retryWhen、delayWhen 等操作符，以及捕获并替代策略、捕获并重抛策略、重试策略等错误处理方法。

<!-- more -->

# Observable 中的约定与错误处理

为了理解 RxJS 中的错误处理，我们需要先知道尽管数据流可以不发送值，也可以发送多个值，但 **一个数据流（stream）只能输出一次错误**。我们做出如此的约定，因为在实践中我们所观察（observe）的所有的流都是这样工作的，比如可能发生错误的网络请求等。

一个流可以顺利完成（complete），这意味着：

- 一个流结束了他的生命周期，在此期间没有发生任何错误
- 结束之后，这个流将不会再发出别的值

一个流也可以报错（error out）：

- 流带着一个错误结束了他的生命周期
- 当错误被抛出之后，这个流将不会再发出别的值

注意 **完成** 和 **报错** 是互斥的：

- 当流完成之后，他不能再报错
- 当流报错之后，他不能再完成

同样需要注意流没有义务去“完成”或者“报错”，这两种可能性均是可选的，但是他们中只有一种情况会出现（是不是有点像薛定谔的猫，你永远不知道一个异步请求会成功还是报错(笑)）。

这意味着，根据 Observable 的约定，当某个流报错之后，我们就不能再使用它了。现在，你一定在考虑一个问题——那么如何来让他从错误中恢复过来呢？

# RxJS 中的 subscribe 方法与 error 回调

现在，我们首先来创建一个流并订阅（subscribe）他。记住 subscribe 方法接受 3 个可选的参数：

- 一个处理成功的函数，每当流发送一个值时就会调用这个函数
- 一个处理错误的函数，只有当错误发生的时候才会调用，这个函数将会接受错误（error）本身作为参数
- 一个处理完成的函数，只有当流完成的时候才会调用

```typescript
@Component({
  selector: 'home',
  templateUrl: './home.component.html'
})
export class HomeComponent implements OnInit {

  constructor(private http: HttpClient) {}

  ngOnInit() {
    const http$ = this.http.get<Course[]>('/api/courses')
    http$.subscribe(
      res => console.log('HTTP response', res),
      err => console.log('HTTP Error', err),
      () => console.log('HTTP request completed.')
    )
  }
}
```

## 完成行为的举例

当这个流没有报错，你可以在控制台看到：

```
HTTP response { payload: Array(9) }
HTTP HTTP request completed.
```

这个 HTTP 流只发送了一个值，随即结束，这意味着他没有发生错误。

但如果他抛出一个错误会怎么样呢？在这种情况下，我们可以在控制台中看到这样的东西：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-1.png)

此时流并没有发出任何值，并且立即报错。在报错之后，他没有执行完成回调。

## subscribe 方法中的 error 回调函数的局限性

有时候给 subscribe 传递 error 回调已经满足了我们处理错误的需求，但这种错误处理的方法是有局限性的。比如我们不能从错误信息中恢复过来，也不能发送另一个备用的值（fallback value）来替代我们原先希望从后端获取的值。

# catchError 操作符

在同步编程中，我们可以使用 `try` 语句来包裹代码，用 `catch` 来获取并处理可能发生的错误。

```typescript
try {
  // 同步操作
  const httpResponse = getHttpResponseSync('/api/courses')
}
catch(error) {
  // 处理错误
}
```

这个机制非常强大，这样我们就可以在一个地方处理 `try/catch` 代码块中发生的任何错误了。

但问题在于，Javascript 中很多操作都是异步的，HTTP 请求就是其中的一个。

RxJS 则通过 `catchError` 操作符提供了一些类似的功能。

## catchError 是如何工作的？

与其他的 RxJS 运算符一样，`catchError` 只是一个输入 Observable，然后输出 Observable 的函数。

每次调用 `catchError` 的时候，需要传给他一个错误处理函数（error handling function）。

`catchError` 操作符将可能报错的 Observable 作为输入，然后开始在他输出的 Observable 中发出输入的 Observable 的值。

如果没有错误发生，他输出的 Observable 跟输入的 Observable 将以同样的方式工作。

## 当错误发生时会怎样？

然而，当错误发生时，`catchError` 逻辑开始生效。`catchError` 操作符将会拿到这个错误，并将其传递给错误处理函数。

这个函数需要返回一个 Observable，他返回的 Observable 将会代替出错的流。

显然根据 Observable 约定，我们不能在使用已经出错的输入 `catchError` 的那个流。

错误处理函数返回的替代的 Observable 将会随即被订阅（subscribe）并且它发出的值将会代替已经报错的输入的 Observable。

# 捕获并替代策略（The Catch and Replace Strategy）

来看一个 `catchError` 用来提供替代的 Observable 并发出备用值的例子：

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    catchError(err => of([]))
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

我们可以将「捕获并替代策略」分解如下：

- 传递给 `catchError` 操作符一个错误处理函数
- 这个错误处理函数并不会立即调用，通常也不会调用
- 只有输入 `catchError` 的 Observable 发生错误时，才会调用错误处理函数
- 如果输入的流发生错误，错误处理函数将会返回一个用 `of([])` 构造的 Observable
- 这个 `of()` 函数只会返回一个值（`[]`），然后完成
- 错误处理函数返回的恢复 Observable（`of([])`）将会被 `catchError` 操作符订阅（subscribe）
- `catchError` 将会把恢复 Observable 的值作为替代值发出

最终，`http$` Observable 将不会报错，我们可以在控制台看到：

```
HTTP response []
HTTP request completed.
```

我们可以看到，`subscribe()` 中的错误处理回调不再被调用，而是发生了下面的操作：

- 一个空数组 `[]` 被发出
- `http$` Observable 随即结束

尽管最初的 `https$` 确实报错了，但替代的 Observable 为 `http$` 的订阅者（subscriber）提供了一个默认的备用值（`[]`）。

注意我们也可以在返回替代的 Observable 之前做一些其他的错误处理。

这就是「捕获并替代策略」，接下来我们看看如何使用 `catchError` 来重新抛出一个错误，而不是提供替代的值。

# 捕获并重抛策略（The Catch and Rethrow Strategy）

注意上面我们通过 `catchError` 提供的替代 Observable 也可能会报错——就像其他任何 Observable 一样。

此时，错误将会被广播给 `catchError` 输出的 Observable 的订阅者。

这种错误广播的行为给了我们一种在本地处理错误之后，重新抛出被 `catchError` 捕获的错误的机制。我们可以这样搞：

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    catchError(err => {
      console.log('Handling error locally and rethrowing it...', err)
      return throwError(err)
    })
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

## 拆解捕获并重抛策略

来一步步拆解这个「捕获并重抛策略」：

- 像之前一样，我们正在捕获一个错误，并且返回一个替代的 Observable
- 但这次，我们在 `catchError` 函数中对这个错误进行本地处理，而不是提供一个像 `[]` 这样的替代值
- 在这个案例中，我们只是简单地在控制台输出错误，但是我们也可以增加任何我们需要的错误处理逻辑，比如向用户展示错误信息等
- `throwError` 创建了一个不会发出任何值的 Observable，并且会立即报出与 `catchError` 捕获的错误一样的错
- 这意味着 `catchError` 输出的 Observable 也会报与输入的 Observable 一样的错
- 这意味着我们终于成功地将 `catchError` 输入的 Observable 的最初的报错 **重新抛给** 输出的 Observable

我们可以在控制台得到：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-2.png)

# 在 Observable 链条中使用多次 catchError

注意我们可以在 Observable 链条中的不同节点根据需要多次使用 catchError，他们可以使用不同的策略。

比如我们可以在 Observable 链条上游中捕获错误，处理并且重抛他，然后在下游捕获同样的错误（这次我们提供一个替代的值，不再重抛他）：

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    map(res => res['payload']),
    catchError(err => {
      console.log('Handling error locally and rethrowing it...', err)
      return throwError(err)
    }),
    catchError(err => {
      console.log('caught rethrown error, providing fallback value')
      return of([])
    })
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-3.png)

我们可以看到，错误事实上最开始被重抛了，但错误并没有到达 subscribe 的错误处理函数，而是发出了 `[]` 这个替代值。

# Finalize 操作符

除了错误处理中的 `catch` 代码块之外，同步 Javascript 语法还提供了 `finally` 代码块来包裹一定会被执行的代码。

`finally` 代码块在释放昂贵资源的时候特别有用，比如关闭网络请求或释放内存。

与 `catch` 代码块中的代码不同，`finally` 代码块中的的执行不受报错的影响。

```typescript
try {
  // 同步操作
  const httpResponse =  getHttpResponseSync('/api/courses')
}
catch {
  // 处理错误，只有当错误发生的时候才会执行
}
finally {
  // 不管什么情况下都会执行
}
```

RxJS 也提供了像 `finally` 一样功能的操作符——`finalize` 操作符。

> 由于是 `finally` 是保留字，因此不能使用。

## Finalize 操作符的例子

就像 `catchError` 操作符一样，为了让不同的资源正确释放，我们可以在 Observable 链条中的不同位置添加多个 `finalize`：

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    map(res => res['payload']),
    catchError(err => {
      console.log('caught mapping error and rethrowing', err)
      return throwError(err)
    }),
    finalize(() => console.log('first finalize() block executed')),
    catchError(err => {
      console.log('caught rethrown error, providing fallback value')
      return of([])
    }),
    finalize(() => console.log('second finalize() block executed'))
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-4.png)

注意 `finalize` 代码块在 subscribe 成功回调和完成回调之后执行

# 重试策略（The Retry Strategy）

除了重抛错误与提供备用值以外，我们还可以重试已经报错的 Observable。

我们知道，一旦流报错，我们不能再恢复他，但我们可以重新从流的源头订阅一次，并创建一个新的流。

它是这样工作的：

- 我们拿到一个输入的 Observable，然后订阅（subscribe）他，创建一个新的流
- 如果流没有报错，我们让他的结果在输出的时候展现
- 但如果流报错了，我们需要去重新订阅这个输入的 Observable，然后创建一个新的流

## 什么时候重试？

一个大问题是，我们什么时候重新订阅这个输入的 Observable，然后重新执行？

- 马上重试？
- 等一小会儿，期待着这个问题被解决，然后再重试？
- 只重试限定的次数，然后报错？

为了解决这些问题，我们需要一个辅助的 Observable，我们叫他「通知者」（Notifier Observable），他会决定什么时候重试。

作为整个「重试策略」的核心 `retryWhen` 将会使用这个「通知者」。

# RxJS retryWhen 操作符的子弹图

为了理解 `retryWhen` Observable 的工作机制，我们可以看看他的子弹图：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-5.png)

注意被重试的 Observable 是第二条线的 Observable 1-2，而不是第一条线的 Observable。

第一条线的发出 r-r 值的 Observable 是「通知者」，他会决定什么时候重试。

## 拆解 retryWhen 是如何工作的

- Observable 1-2 被订阅，然后他的值被立即反射在 `retryWhen` 的输出 Observable 中
- 甚至在 Observable 1-2 结束之后，他仍然可以被重试
- 订阅者在 Observable 1-2 结束之后发出值 `r`
- 订阅者发出的值可以是任何形式的（在这里就是 `r`）
- 重要的是 `r` 发出的时机，因为这就是 Observable 1-2 重试的时机
- Observable 1-2 被 `retryWhen` 重新订阅，他的值又会被重新反射到 `retryWhen` 的输出 Observable 中
- 然后，订阅者最终还是结束了
- 在这个时候，正在进行中的 Observable 1-2 重试也被提早结束，意味着只发出了 1，而不会发出 2

我们可以看到，`retryWhen` 只是在观察着发出值的时候重试输入的 Observable。

既然我们理解了 `retryWhen` 是如何工作的，我们现在来创建一个「通知者」

# 创建一个通知者（Notification Observable）

我们需要在传递给 `retryWhen` 操作符的函数中直接创建一个通知者。这个函数将一个 Errors Observable（发出输入的 Observable 的错误的 Observable）作为参数。

然后通过订阅这个 Errors Observable，我们可以准确地知道什么时候发生了错误。我们现在来看看如何实施一个「立即重试策略」

# 立即重试策略（Immediate Retry Strategy）

为了在错误发生之后立即重试，我们只需要返回这个 Errors Observable 即可。

在这个案例中，我们使用 `tap` 操作符来打 log，因此 Errors Observable 还是不变的：

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    tap(() => console.log("HTTP request executed")),
    map(res => Object.values(res['payload'])),
    shareReplay(),
    retryWhen(errors => {
      return errors
        .pipe(
          tap(() => console.log('retrying...'))
        )
    })
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

我们从 `retryWhen` 函数返回的 Observable 就是「通知者」（Notification Observable）。

他发出的值并不重要，重要的是只有他发出的时机，因为这就是重试的时机。

## 立即重试策略的控制台输出内容

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-6.png)

我们可以看到，HTTP 请求最开始错误了，但是后来尝试了一次，并且第二次的请求成功了。

可以看看两次尝试之间的延迟：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-7.png)

# 延迟重试策略（Delayed Retry Strategy）

让我们来实施另一个错误恢复策略，比如在错误发生之后等待 2 秒再重试。

这个策略在一些从一些特定的错误（比如服务器拥塞导致的请求失败）上恢复的时候特别有效。

在某些错误间歇性发生的案例中，我们可以简单地在一小段延迟之后重试，然后第二次请求可能就没问题了。

# 计时器（Timer Observable）创建函数

为了实施延迟重试策略，我们需要创建一个错误发生之后 2 秒再发出值的通知者。

接下来我们来尝试用 `timer` 来创建一个通知者，这个 `timer` 函数接受两个值：

- 一个在第一个值发出之前的初始延迟
- 一个我们希望周期性发出值时的周期性延迟

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-8.png)

我们可以看到，第一个值 0 在 3 秒之后才发出，然后每隔 1 秒发出一个值。

注意第二个参数是可选的，比如在我们希望在只发出一个值 0 之后就结束。

接下来我们来看看如何如何结合 `retryWhen` 和 `delayWhen` 操作符。

# delayWhen 操作符

一件很重要的事情就是，定义通知者的函数只会被调用一次，因此我们只有一次机会来定义这个确定何时重试的通知者。

我们将要获取 Errors Observable，然后应用在 `delayWhen` 操作符上来定义通知者。

来在这张子弹图上想一想，源 Observable a-b-c 是 Errors Observable，会随着时间发出失败的 HTTP 错误：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-9.png)

## delayWhen 操作符的拆解

我们通过这张图来学习一下：

- 输入的 Errors Observable 的每个值都会被延迟之后展示在输出 Observable 中
- 每个值的延迟可以不同，并且将会被用非常灵活的方式创建
- 为了确定延迟，我们将会每个输入的 Errors Observable 的值调用传递给 `delayWhen` 的函数（也叫作持续时间选择函数，duration selector function））
- 这个函数将会发出一个将会决定每个值发出之后的延迟
- a-b-c 中的每一个值都有自己的持续时间选择器（duration selector Observable），他最终会发出一个值（可以是任何值），然后结束
- 当三个持续时间选择器中的每个发出值的时候，对应的输入值 a-b-c 就会在最后的输出中展示
- 注意值 `b` 在 `c` 之后出现，这是很正常的
- 因为 `b` 持续时间选择器在 `c` 持续时间选择器之后发出值

## 实施延迟重试策略

让我们现在来把这些都放到一起，看看我们怎样按顺序在 2 秒后重试每一个发生错误的 HTTP 请求。

```typescript
const http$ = this.http.get<Course[]>('/api/courses')
http$
  .pipe(
    tap(() => console.log("HTTP request executed")),
    map(res => Object.values(res['payload'])),
    shareReplay(),
    retryWhen(errors => {
      return errors
        .pipe(
          delayWhen(() => timer(2000)),
          tap(() => console.log('retrying...')),
        )
    })
  )
  .subscribe(
    res => console.log('HTTP response', res),
    err => console.log('HTTP Error', err),
    () => console.log('HTTP request completed.')
  )
```

来拆解一下：

- 传给 `retryWhen` 的函数只会调用一次
- 我们在那个函数中返回一个 Observable，他在任何需要重试的时候就会发送值
- 每当出现一个错误，`delayWhen` 操作符就会通过调用 `timer` 函数来创建一个持续时间选择器（duration selector Observable）
- 这个持续时间选择器将会在 2 秒后发出 0，然后结束
- 每当这发生的时候，delayWhen Observable 就会知道输入的错误的延迟已经结束了
- 只有当延迟结束之后，错误才会在通知者的输出中出现
- 一旦通知者中的值发出，`retryWhen` 操作符就会执行重试

## 延迟策略的控制台输出内容

下面是某个 HTTP 请求，重试了 5 次，并且其中 4 次报错的情况：

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-10.png)

![](https://s3-us-west-1.amazonaws.com/angular-university/blog-images/rxjs-error-handling/rxjs-error-handling-11.png)

可以看到与预期一样，重试在错误发生 2 秒之后才会进行。
