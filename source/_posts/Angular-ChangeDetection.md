---
title: 翻译：Angular 如何检查变化
date: 2021-05-17 18:56:06
tags:
  - 入门
  - 文档
categories:
  - [前端, Angular]
---

翻译文章 [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/) 。

<!-- more -->

* 下文中 CD 指 Change Detection，即变化检查；CDr 指 Change Detector，即变化检查器

# 目录

- CD 是如何进行的？
- Angular 的变化检查器是什么样的？
- 默认的 CD 机制是怎样进行的
- 开启/关闭 CD ，手动触发他
- 避开 CD 循环：生产模式和开发模式
- `OnPush` CD 模式究竟做了什么？
- 使用 Immutable.js 来简化 Angular 应用的构建
- 总结

如果想要了解更多关于 OnPush CD 机制，请查阅 [这篇文章](https://blog.angular-university.io/onpush-change-detection-how-it-works) 。

# CD 是如何实现的？

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

Zone 其实就是一个在多个 JavaScript 虚拟机执行回合之中幸存下来的执行上下文——这是一个可以用来为浏览器增加更多功能的很普遍的机制。Angular 内部使用 Zones 来触发 CD ，但 Zone 也可以用来做应用分析（比如内存占用、程序复杂度等）、跨越不同的 JavaScript 虚拟机回合进行长堆栈跟踪。

# 浏览器异步 API 支持

为了支持 CD ，下面这些常用的浏览器机制也被修改了：

- 所有的浏览器事件（click、mouseover、keyup 等）
- `setTimeout()` 和 `setInterval()`
- Ajax HTTP 请求

事实上，为了无感知地触发 Angular CD ，Zone.js 也修改了很多别的浏览器 API，比如 Websocket。可以看看 Zone.js 的 [一段测试代码](https://github.com/angular/zone.js/tree/master/test/patch) 看看现在支持了哪些 API。

这个机制的局限性在于：如果出于某些原因，Zone.js 不支持某个浏览器的异步 API，那么 CD 就不会被触发。比如 IndexedDB 的回调。

上面解释了 CD 是怎样触发的，但是触发之后他到底是怎样运作的呢？

# CD 树

在应用的启动阶段，每个 Angular 组件都与一个 CDr 建立关联。比如下面的 `TodoItem` 组件：

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

# TodoItem 的 CDr 长什么样子？

事实上，我们可以在运行时看到 CDr 长什么样子！我们只需要在 Todo class 里面加一些代码来在访问他的属性的时候 [打一个断点](https://github.com/jhades/blog.angular-university.io/blob/master/ng2-change-detection/src/todo.ts#L11) 。

到达断点的时候，我们可以跟着调用栈看到 CDr：

![](https://raw.githubusercontent.com/jhades/blog.angular-university.io/master/ng2-change-detection/images/change.jpg)

别担心，你不需要去 debug 这些代码。这里也没有引入什么魔法，他只是一个在应用的启动阶段创建的简单的 Javascript 方法。但是他到底做了什么呢？

# 默认的 CD 机制是怎么工作的？

这个方法最开始看起来可能非常奇怪，还有一些奇怪名字的变量。但是我们来仔细看看，可以注意到他在做一些非常简单的事情：他会比较每个表达式中使用的属性现在和之前的值，如果一个属性现在和之前的值不一样，他会将 `isChanged` 设为 `true`。

它使用了 `looseNotIdentical()` 来比较两个值，[这个方法](https://github.com/angular/angular/blob/50548fb5655bca742d1056ea91217a3b8460db08/modules/angular2/src/facade/lang.ts#L367) 使用了 `===` 运算符，并且对 `NaN` 进行了一些特殊处理。

```typescript
function looseIdentical(a, b): boolean {
  return a === b || typeof a === "number" && typeof b === "number" && isNaN(a) && isNaN(b)
}
```

# 对于嵌套对象 `owner` 又怎么样呢？

我们可以看到在 CDr 的代码中，`owner` 中的对象也被检查了，但是只检查了 `firstName`，并没有检查 `lastName`。

这是因为 `lastName` 并没有被 template 使用！——同样地，Todo 中的 `id` 属性也没有被检查。

对此，我们可以说：

> 默认情况下，Angular CD 是检查 template 表达式中的值是否变化。他会检查所有的组件。

我们也可以总结一下：

> 默认情况下，Angular 不会做对象的深比较来检查便捷化，他只会关心在 template 中使用的属性。

# 为什么 CD 默认这样工作？

Angular 的主要目标之一就是变得更透明和易用，因此框架的使用者不需要为了更有效率地使用它而去进行很冗长的 debug 或者了解他的内部机制。

如果你熟悉 AngularJs，想一下 `$digest()` 和 `$apply()` 以及所有使用/不使用他们的时候的坑。Angular 的主要目标之一就是避免他们。

# 为什么不比较引用？

事实上，Javascript 的对象都是可变的，Angular 想要对此给予全力的支持。

想象一下如果 Angular 默认 CD 机制是基于组件输入的引用的比较（而不是默认的机制）。这样的话即便是像 TODO 这样简单的应用都需要一些技巧来创建：开发者需要非常小心地来创建一个新的 Todo，而不是只是更新属性。

但就像我们将要看到的那样，如果有必要的话，也可以自定义 Angular 的CD 。

# 性能如何？

注意 TodoList 组件的CD 器是怎样创建对 `todos` 属性的显式引用的。

另一个方法是动态地遍历组件的属性，让代码具有普遍性（generic）——而不是为了这个组件特异化（specific）。这样我们就不用在每次组件创建的时候都创建一个CD 器了！所以应该怎么做呢？

# 先看看虚拟机里面是什么样的

上面说的一切都与 Javascript 虚拟机有关。进行动态比较的代码并不能被虚拟机即时编译器（VM just-in-time compiler）优化为本地代码（native code）。

特异化的代码对每个组件输入的属性有非常明确的访问，他们就像我们自己写的代码一样，而且虚拟机可以非常轻松地将他们转换为本地代码。

用生成的特异化 CDr 的结果就是：会非常快、可预测和易读。

接下来我们来讨论一下性能，有办法可以优化 CD 吗？

# `OnPush` CD 模式

如果我们的 Todo 列表非常的大，我们可以配置这个 `TodoList` 组件，使用 `OnPush` 策略，让他只有在 Todo 列表变化的时候才会更新自己。

```typescript
@Component({
  selector: "todo-list",
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: "...",
})
export class TodoList {}
```

然后在整个列表外面加两个按钮：

- 一个按钮直接修改 TodoItem，来改变他的状态
- 一个按钮给整个 TodoList 增加一个新的 TodoItem

```typescript
@Component({
  selector: "app",
  template: `
  <div>
    <todo-list [todos]="todos"></todo-list>
  </div>
  <button (click)="toggleFirst()">Toggle First Item</button>
  <button (click)="addTodo()">Add Todo to List</button>
  `,
})
export class App {
  todos: Array = initialData

  constructor() {}

  toggleFirst() {
    this.todos[0].completed = ! this.todos[0].completed
  }

  addTodo() {
    let newTodos = this.todos.slice(0)
    newTodos.push(
      new Todo(1, "TODO 4", false, new Owner("John", "Doe"))
    )
    this.todos = newTodos
  }
}
```

来看看他们不同的表现：

- 第一个按钮 "Toggle First Item" 没有起作用！因为 `toggleFirst()` 函数直接改变了列表的第一个元素。`TodoList`。`TodoList` 无法检测到这个这个变化，因为输入的 `todos` 引用没有发生变化
- 第二个按钮生效了！注意 `addTodo()` 创建了一个 todo list 的副本，然后在这个副本上增加一个项，最后用这个副本替代掉原来的 todos。因为输入的 todos 的引用发生了变化，这就触发了检查。
- 第二个按钮中，直接修改这个 todo list 是不会生效的，我们需要一个新的 list

# `OnPush` 真的只是检查引用吗？

这不是问题，就算你将 `TodoItem` 也改成 `OnPush` 模式，你仍然可以做到只是点击一个 todo 就 toggle 他。

因为 `OnPush` 不只是在组件输入变化的时候进行检查，组件触发的事件同样会触发CD 。

正如 Victor Savkin 在他的博客中所说：

> 如果使用 OnPush 检查器的时候，框架会在他输入的属性变化的时候、他发出事件的时候、Observable 触发事件的时候，检查 OnPush 组件

尽管 `OnPush` 可能会使得性能更好，在使用可变对象时，他的引入会带来更高的复杂度。这会引发一些很难找到原因和重现的 BUG。但是也有可以灵活使用 `OnPush` 的方法。

# 使用 `Immutable.js` 来简化 Angular 应用的构建

如果我们只用不可变的对象和列表来构建应用，我们显然就可以在任何地方使用 `OnPush` 了，并且不会导致 CD 的 BUG。这是因为“修改”不可变对象的唯一方法是创建一个新的对象来替代它。 对于不可变对象，我们可以保证：

- 一个新的不可变对象永远都会触发 `OnPush` 的 CD
- 不可能产生因为忘记创建一个对象的新副本而引发的错误

用 Immutable.js 库是个不错的选择，这个库提供了构建应用的基础，比如不可变对象、Map、列表等。

# 避免 CD 循环：生产模式 vs 开发模式

Angular CD 的一个重要特性就是，与 AngluarJS 不同，他强制使用单向数据流：当我们的 controller 更新的时候，CD 会运行并更新 view。

但是更新 view 并不会触发变化（变化又会触发新的视图更新，在 AngularJS 中这被称为 digest cycle）。

# 怎样在 Angular 中触发 CD 循环？

```typescript
@Component({
  selector: 'todo-list',
  directives: [TodoItem],
  template: `<ul>
                <li *ngFor="#todo of todos;">
                    <todo-item [todo]="todo" (toggle)="onToggle($event)"></todo-item>
                </li>
           </ul>
           <button (click)="blowup()">Trigger Change Detection Loop</button>`
})
export class TodoList implements AfterViewChecked {

  @Input()
  todos: Array<Todo>;

  @Input()
  callback: Function;

  @Output()
  addTodo = new EventEmitter<Object>();

  clicked = false;

  onToggle(todo) {
    console.log("toggling todo..");
    todo.completed = !todo.completed;
  }

  blowup() {
    console.log("Trying to blow up change detection...");
    this.clicked = true;
    this.addTodo.emit(null);
  }

  ngAfterViewChecked() {
    if (this.callback && this.clicked) {
      console.log("changing status ...");
      this.callback(Math.random());
    }
  }

}
```

一种情况是使用生命周期钩子：比如在 `TodoList` 组件中，我们可以在 `ngAfterViewChecked` 中通过回调函数去改变另外一个组件绑定的值，这时候就会得到一个警告：

```text
EXCEPTION: Expression '{{message}} in App@3:20' has changed after it was checked
```

这个错误只有在开发模式下会出现，在生产模式下将不会抛错，这个 issue 也不会被检测到。

# CD issues 会很频繁吗？

事实上我们不应该触发 CD 循环，以防万一，我们我们在开发阶段应该一直使用开发模式来避免这个问题。 因为 Angular 在开发模式下，总是会进行两次 CD（第二次就是用来检测这个问题）；而在生产模式下，CD 只会执行一次。

# 开/关 CD，并手动触发他

在一些特殊的情况下，我们不想触发 CD。设想这样一种情况，我们从后端通过 Websocket 拿到了大量的数据，我们想要每 5 秒钟更新 UI 的一部分。为了达成这个目的，我们需要在最开始的时候注入 CDr：

```typescript
class MyComponent {
  constructor(
    private ref: ChangeDetectorRef,
  ) {
    ref.detach()
    setInterval(() => {
      this.ref.detectChanges()
    }, 5000)
  }
}
```

我们最开始 detach 了 CDr（这会关闭自动的 CD），然后每 5 秒通过 `detectChanges` 手动触发一次。

# 总结

Angular CD 是一个框架内置的特性，用于保证组件数据和 HTML 模板视图的同步。

CD 会检测常规的浏览器事件（比如点击）、HTTP 请求，以及其他类型的事件，并决定每个组件的视图是否需要更新。

有中不同的 CD：

- 默认 CD：对于组件树种所有的组件，Angular 通过比较 template 中表达式的在事件前后的值，来决定视图是否更新
- `OnPush` CD：他会检查是否有新的数据确实地被推到了组件中（可以通过组件 input 或者使用 async pipe 订阅了的 Observable）

Angular **默认** CD 机制与 AngularJS 非常接近，他会检查 template 表达式在浏览器事件前后的值来看是否有变化。他对 **所有** 的组件都会这么做，但也有几点不同：

- 其一是 Angular 中没有 CD 循环（AngularJS 中叫 digest cycle）。这允许我们在只看 template 和 controller 的情况下能推导出每个组件。
- 其二是由于 CDr 是被构建出来的，Angular 中的 CD 机制会快很多。
- 第三是与 AngularJS 不同，Angular 中的 CD 机制是可定制的。

