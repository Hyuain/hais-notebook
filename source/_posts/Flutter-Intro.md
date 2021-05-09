---
title: Flutter 简单上手
date: 2021-04-24 10:13:00
tags:
  - 入门
categories:
  - [跨端, Flutter]
mathjax: true
---

这几天简单了解了一下 Flutter，以下笔记是阅读文档的笔记。

<!-- more -->

# UI

## Widget

这是公式：`$ x_i^2 $`
这是公式2：

$$
x_i^2
$$


Widget 像其他 Web 框架的 Component、Template、DOM，用来描述 UI。

> 当 widget 的状态改变时，它会重新构建其描述（展示的 UI），框架则会对比前后变化的不同，以确定底层渲染树从一个状态转换到下一个状态所需的最小更改。

### Root Widget

往 `runApp()` 里面传入一个 Widget 就可构建一个最小的 Flutter 应用，传入的这个 Widget 叫做 `Root Widegt`

```dart
void main() {
  runApp(
    Center(
      child: Text(
        'Hello, world!',
        textDirection: TextDirection.ltr,
      ),
    ),
  );
}
```

{% note warning %}
Root Widget 会被强制指定铺满整个屏幕。
{% endnote %}

## 一些基础 Widget

- `Text`：创建带样式的文本
- `Row`, `Column`：用于在水平和垂直方向进行创建类似 flex box 的东西以进行布局
- `Stack`：类似于 CSS 层叠上下文，是按照绘制顺序将 Widget 堆叠在一起。可以将 `Positioned` 作为 child，将其相对于 `Stack` 进行定位。
- `Container`：类似于 `<div></div>` 创建一个矩形，可以用 `BoxDecoration` 来加背景、边框、阴影等，还可以指定 margin、padding、变换等。
- `GestureDetector`：本身不可见，但是可以探测到内部组件的手势事件，比如点击、拖动、缩放。

## 创建一个自己的 Widget

可以创建一个 class，他继承了 `StatelessWidget`，这个 class 就是一个 Widget。

```dart
class MyAppBar extends StatelessWidget {
  MyAppBar({required this.title});

  // Fields in a Widget subclass are always marked "final".

  final Widget title;

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 56.0, // in logical pixels
      padding: const EdgeInsets.symmetric(horizontal: 8.0),
      decoration: BoxDecoration(color: Colors.blue[500]),
      // Row is a horizontal, linear layout.
      child: Row(
        // <Widget> is the type of items in the list.
        children: <Widget>[
          IconButton(
            icon: Icon(Icons.menu),
            tooltip: 'Navigation menu',
            onPressed: null, // null disables the button
          ),
          // Expanded expands its child
          // to fill the available space.
          Expanded(
            child: title,
          ),
          IconButton(
            icon: Icon(Icons.search),
            tooltip: 'Search',
            onPressed: null,
          ),
        ],
      ),
    );
  }
}
```

## `StatelessWidget` 和 `StatefulWidget`

- `StatelessWidget`：无状态 widget。接收的参数来自于它的父 widget，这些参数被储存在 final 成员变量中。当 widget 需要被 build() 时，就是用这些存储的变量为创建的 widget 生成新的参数。
- `StatefulWidget`：有状态 widget。他有一些自己管理的状态，可以自行做出一些响应用户操作的变化。

```dart
import 'package:flutter/material.dart';

class Counter extends StatefulWidget {
  // 这个 class 是用来接收父级传过来的属性的（就像无状态 Widget 一样，他们通常被以 final 的形式存储）
  // 在这个例子中，这个 class 没有接收任何参数

  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _counter = 0;

  void _increment() {
    setState(() {
      // 调用 setState 会告诉 Flutter：State 发生了变化。
      // 这会调用下面的 build 函数来更新视图
      // 如果不调用 setState，视图将不会更新
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    // 每次 setState 的时候都会调用这个函数
    // Flutter 已经对 build 过程进行了优化，所以可以直接重新调用 build，而不必关心局部 Widget 的更新
    print("build");
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        ElevatedButton(
          // 点击按钮的时候调用 _increment
          onPressed: _increment,
          child: Text('Increment'),
        ),
        SizedBox(width: 16),
        Text('Count: $_counter'),
      ],
    );
  }
}
```

## 父子 Widget 的状态传递

> 简单来说，就像 React。

父 -> 子：值
子 -> 父：回调函数

```dart
// 一个用于展示 count 的无状态组件
class CounterDisplay extends StatelessWidget {
  CounterDisplay({required this.count});

  final int count;

  @override
  Widget build(BuildContext context) {
    return Text('Count: $count');
  }
}

// 一个用于增加 count 的无状态组件
class CounterIncrementor extends StatelessWidget {
  CounterIncrementor({required this.onPressed});

  final VoidCallback onPressed;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text('Increment'),
    );
  }
}

// 父组件
class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

// 父组件的状态管理部分
class _CounterState extends State<Counter> {
  int _counter = 0;

  void _increment() {
    setState(() {
      ++_counter;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        // 这是刚刚定义的用于增加 count 的无状态组件，通过回调函数将事件发给父级控制
        CounterIncrementor(onPressed: _increment),
        SizedBox(width: 16),
        // 这是刚刚定义的用于显示 count 的无状态组件，传给他 _count，让他只负责显示
        CounterDisplay(count: _counter),
      ],
    );
  }
}
```

可以看 [官方文档的购物车例子](https://flutter.cn/docs/development/ui/widgets-intro#bringing-it-all-together)

## 状态的生命周期

- `initState()`：在 StatefulWidget 上调用 createState() 之后，框架将新的状态对象插入到树中，然后在状态对象上调用 initState()。 State 的子类可以重写 initState 来完成只需要发生一次的工作。例如，重写 initState 来配置动画或订阅平台服务。实现 initState 需要调用父类的 super.initState 方法来开始。

- `dispose()`：当不再需要状态对象时，框架会调用状态对象上的 dispose() 方法。可以重写dispose 方法来清理状态。例如，重写 dispose 以取消计时器或取消订阅平台服务。实现 dispose 时通常通过调用 super.dispose 来结束。



## 布局

Flutter 中的布局也是使用 Widget 实现的。可以在 [Flutter 布局 Widget](https://flutter.cn/docs/development/ui/widgets/layout) 中找。

# 状态管理

## 声明式 UI

- 命令式（比如 Web）：如果我们要操作 UI，通常需要通过 `getElementBuyId` 之类的 API 先获取 DOM 中元素的实例和操作权，然后再调用相关的方法来修改他的颜色等属性。
- 声明式（Flutter）：视图本身（Widget）是不可变的，通常在配置好之后，我们不能改变 Widget，如果需要改变 UI，需要通过 `setState()` 之类的方法构建一个新的 Widget。

## 短时状态（Ephemeral State）与应用状态（App State）

- 状态
  - 广义：这个应用运行时存在于内存中的所有内容。包括应用中用到的资源，所有 Flutter 框架中有关用户界面、动画状态、纹理、字体以及其他等等的变量。
  - 狭义：是当任何时候需要**重建你的用户界面**时所需要的**数据**。因为很多广义的状态不需要我们管理（比如纹理），框架本身会替我们管理。
- 短时状态：有时也称 *用户界面(UI)状态* 或者 *局部状态*。是可以完全包含在 **一个独立 widget** 中的状态。
  - 一个 PageView 组件中的当前页面
  - 一个复杂动画中当前进度
  - 一个 BottomNavigationBar 中当前被选中的 tab
- 应用状态：有时也称 **共享状态**。是指在应用的多个部分之间共享的一个非短时状态，并且在用户会话期间会保留这个状态。
  - 用户选项
  - 登录信息
  - 一个社交应用中的通知
  - 一个电商应用中的购物车
  - 一个新闻应用中的文章已读/未读状态

> 简单来说。
> 短时状态：用 setState 管理，别人不需要关心你的状态，类似于 React 中的 setState。
> 应用状态：可以用状态管理框架（比如 Redux）来管理，多个部分会共享这个状态。

## Provider

Provider 可以用于进行管理应用状态。

### ChangeNotifier

{% note warning %}
ChangeNotifier 实际上是在 `flutter:foundation` 中的，不依赖其他的库，也不一定要与 Provider 一起使用。
{% endnote %}

```dart
/// 创建一个类，他继承 ChangeNotifier，负责管理状态
/// 这里的例子是用来管理购物车中的商品
class CartModel extends ChangeNotifier {
  /// 内部的私有变量，用来保存数据
  final List<Item> _items = [];

  /// 向外界暴露出一个不可更改的 ListView
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// 向外界暴露出我们计算出的商品总价
  int get totalPrice => _items.length * 42;

  /// 暴露出 add 函数，用于往购物车增加商品
  void add(Item item) {
    _items.add(item);
    // 关键！调用 notifyListeners() 告诉正在监听这个 model 的 widget，让他们 rebuild
    notifyListeners();
  }

  /// 暴露出 removeAll 函数，用于清空所有商品
  void removeAll() {
    _items.clear();
    // 关键！调用 notifyListeners() 告诉正在监听这个 model 的 widget，让他们 rebuild
    notifyListeners();
  }
}
```

### ChangeNotifierProvider

`ChangeNotifierProvider` widget 可以向其子孙节点暴露一个 `ChangeNotifier` 实例。它属于 provider package。

需要将 `ChangeNotifierProvider` 放在所有需要访问他的 widget 之上。

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      /// 定义了一个 builder 来创建 CartModel 实例，ChangeNotifierProvider 不会重复实例化 CartModel
      /// 如果该实例已经不会再被调用， ChangeNotifierProvider 也会自动调用 CartModel 的 dispose() 方法。
      create: (context) => CartModel(),
      child: MyApp(),
    ),
  );
}
```

可以使用 `MultiProvider` 来提供多个状态：

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => CartModel()),
        Provider(create: (context) => SomeOtherClass()),
      ],
      child: MyApp(),
    ),
  );
}
```

### Consumer

可以通过 Consumer 来获取到数据

```dart
Widget build(BuildContext context) {
  // 通过 Consumer 来获取到数据
  return Consumer<CartModel>(
  /// 当 ChangeNotifier 发生变化（调用 notifyListener()）的时候会调用 builder。
  /// builder 的第一个参数是 context，跟其他的 build 方法一样
  /// builder 的第二个参数是 ChangeNotifier 的实例，也就是 CartModel 的实例
  /// builder 的第三个参数是 child，用于优化目的。
  /// 如果 Consumer 下面有一个庞大的子树（SomeExpensiveWidget），当 Model 发生改变的时候，该子树（SomeExpensiveWidget） 并不会 改变，那么你就可以仅仅创建它一次，然后通过 builder 获得该实例。
    builder: (context, cart, child) => Stack(
        children: [
          // 不需要每次都 rebuild 的庞大子树
          child,
          Text("Total price: ${cart.totalPrice}"),
        ],
      ),
    // 不需要每次都 rebuild 的庞大子树
    child: SomeExpensiveWidget(),
  );
}
```

一般来讲我们需要把 Consumer 放到尽可能低的位置，这样每次 Model 的变化就不会引起过量的 rebuild

### Provider.of

有时候我们不需要页面更新，又想要用到新的数据，这时候就可以不用 `Consumer`，而是 `Provider.of`：

```dart
Widget build(BuildContext context) {
  // 并且需要将 listen 设置为 false，这样当 notifyListeners 被调用的时候，并不会使 widget 被重构
  Provider.of<CartModel>(context, listen: false).removeAll();
  //...
}
```