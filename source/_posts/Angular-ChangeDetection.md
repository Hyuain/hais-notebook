---
title: 翻译：Angular 如何检查变化
date: 2021-05-17 18:56:06
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, Angular]
---

翻译文章 [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/) 。

<!-- more -->

# 目录

- 变化检查是如何进行的？
- Angular 变化检查器是什么样的？
- 默认的变化检查机制是怎样进行的
- 开启/关闭变化检查，手动触发他
- 避开变化检查循环：生产模式和开发模式
- `OnPush` 变化检查模式究竟做了什么？
- 使用 Immutable.js 来简化 Angular 应用的构建
- 总结

如果想要了解更多关于 OnPush 变化检查机制，请查阅 [这篇文章](https://blog.angular-university.io/onpush-change-detection-how-it-works) 。

# 变化检查是如何实现的？

当组件的数据变化的时候，Angular 可以探测到，并重新渲染视图来相应这次变化。为了理解工作原理，我们需要首先意识到 JavaScript 中整个运行时都是可重写的。我们可以重写类似于 `String` 或 `Number` 等方法。

# 重写浏览器的默认机制

Angular 启动的时候，将一些底层的的浏览器 API（比如浏览器用来注册事件的 `addEventListener`）替换掉，就像这样：

```javascript
// 这是新版的 addEventListener
function addEventListener(eventName, callback) {
  // 调用原生的 addEventListener
  callRealAddEventListener(eventName, function() {
    // 先调用原生的 callback
    callback(...)
    // 再运行 Angular 添加的功能
    var changed = angular.runChangeDetection()
    if (changed) {
      angular.reRenderUIPart()
    }
  })
}
```

新版的 `addEventListener` 会给事件处理器添加更多的功能：除了调用回调函数之外，还让 Angular 可以检测到变化并更新视图。

# 这个底层的运行时替换是怎样工作的？

一个叫 [Zone.js](https://github.com/angular/zone.js/) 的 Angular 自带的库完成了这个工作。

Zone 其实就是一个在多个 JavaScript 虚拟机执行回合之中幸存下来的执行上下文——这是一个可以用来为浏览器增加更多功能的很普遍的机制。Angular 内部使用 Zones 来触发变化检查，但 Zone 也可以用来做应用分析（比如内存占用、程序复杂度等）、跨越不同的 JavaScript 虚拟机回合进行长堆栈跟踪。

# 浏览器异步 API 支持

为了支持变化检查，下面这些常用的浏览器机制也被修改了：

- 所有的浏览器事件（click、mouseover、keyup 等）
- `setTimeout()` 和 `setInterval()`
- Ajax HTTP 请求

事实上，为了无感知地触发 Angular 变化检查，Zone.js 也修改了很多别的浏览器 API，比如 Websocket。可以看看 Zone.js 的 [一段测试代码](https://github.com/angular/zone.js/tree/master/test/patch) 看看现在支持了哪些 API。

这个机制的局限性在于：如果出于某些原因，Zone.js 不支持某个浏览器的异步 API，那么变化检查就不会被触发。比如 IndexedDB 的回调。

上面解释了变化检查是怎样触发的，但是触发之后他到底是怎样运作的呢？

# 变化检查树

在应用的启动阶段，每个 Angular 组件都与一个变化检查器建立关联。比如下面的 `TodoItem` 组件：

```typescript
@Component({
  selector: 'todo-item',
  template: `
    <span (click)="onToggle()" class="todo noselect">
      {{todo.owner.firstName}} - {{todo.description}} - completed: {{todo.completed}}
    </span>
  `
})
export class TodoItem {
  @Input()
  todo: Todo
  
  @Output()
  toggle = new EventEmitter<Object>()
  
  onToggle() {
    this.toggle.emit(this.todo)
  }
}
```

这个组件会接受一个 `Todo` 对象作为输入，并且当 todo 状态发生改变的时候发出一个事件。来搞点花样，[Todo class](https://github.com/jhades/blog.angular-university.io/blob/master/ng2-change-detection/src/todo.ts#L11) 里面还嵌套了一个对象：

```typescript
export class Todo {
  constructor(
    public id: number,
    public description: string,
    public completed: boolean,
    public owner: Owner // 嵌套了 Owner 对象
  ) {}
}
```

`Todo` 有一个属性 `owner`，`owner` 也是一个对象（有属性 `firstName` 和 `lastName`）。

# TodoItem 的变化检查器长什么样子？

事实上，我们可以在运行时看到变化检查器长什么样子！我们只需要在 Todo class 里面加一些代码来在访问他的属性的时候 [打一个断点](https://github.com/jhades/blog.angular-university.io/blob/master/ng2-change-detection/src/todo.ts#L11) 。

到达断点的时候，我们可以跟着调用栈看到变化检查：

![](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)

别担心，你不需要去 debug 这些代码。这里也没有引入什么魔法，他只是一个在应用的启动阶段创建的简单的 Javascript 方法。但是他到底做了什么呢？

# 默认的变化检查机制是怎么工作的？

这个方法最开始看起来可能非常奇怪，还有一些奇怪名字的变量。但是我们来仔细看看，可以注意到他在做一些非常简单的事情：他会比较每个表达式中使用的属性现在和之前的值，如果一个属性现在和之前的值不一样，他会将 `isChanged` 设为 `true`。

它使用了 `looseNotIdentical()` 来比较两个值，[这个方法](https://github.com/angular/angular/blob/50548fb5655bca742d1056ea91217a3b8460db08/modules/angular2/src/facade/lang.ts#L367) 使用了 `===` 运算符，并且对 `NaN` 进行了一些特殊处理。

```typescript
function looseIdentical(a, b): boolean {
  return a === b || typeof a === "number" && typeof b === "number" && isNaN(a) && isNaN(b)
}
```

# 对于嵌套对象 `owner` 又怎么样呢？

我们可以看到在变化检查器的代码中，`owner` 中的对象也被检查了，但是只检查了 `firstName`，并没有检查 `lastName`。

这是因为 `lastName` 并没有被 template 使用！——同样地，Todo 中的 `id` 属性也没有被检查。

对此，我们可以说：

> 默认情况下，Angular 变化检查是检查 template 表达式中的值是否变化。他会检查所有的组件。

我们也可以总结一下：

> 默认情况下，Angular 不会做对象的深比较来检查便捷化，他只会关心在 template 中使用的属性。

# 为什么变化检查默认这样工作？

Angular 的主要目标之一就是变得更透明和易用，因此框架的使用者不需要为了更有效率地使用它而去进行很冗长的 debug 或者了解他的内部机制。

如果你熟悉 AngularJs，想一下 `$digest()` 和 `$apply()` 以及所有使用/不使用他们的时候的坑。Angular 的主要目标之一就是避免他们。

# 那通过引用比较又是怎样的呢？

事实上，Javascript 的对象都是可变的，Angular 想要对此给予全力的支持。

想象一下如果 Angular 默认变化检查机制给予组件输入的引用的比较（而不是默认的机制）。这样的话即便是像 TODO 这样简单的应用都需要一些技巧来创建：开发者需要非常小心地来创建一个新的 Todo，而不是只是更新属性。

但就像我们将要看到的那样，如果有必要的话，也可以自定义 Angular 的变化检查。