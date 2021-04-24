---
title: Flutter 简单上手
date: 2021-04-24 10:13:00
tags:
  - 入门
categories:
  - [跨端, Flutter]
---

这几天简单了解了一下 Flutter，以下笔记是阅读文档的笔记。

<!-- more -->

# UI

## Widget

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

