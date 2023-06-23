---
title: React - Deep Dive
date: 2023-06-23 16:41:41
categories:
  - [前端]
---

与另一篇 React 笔记不同，本文主要是深入讨论 React 的源码与设计理念。

<!-- more -->

Clone React 项目之后，在 README 中找到调试 React 的方法，具体可参照 [How to Contribute](https://legacy.reactjs.org/docs/how-to-contribute.html)。然后按照文中安装依赖并 Build 之后，将 `fixture/packaging/babel-standalone/dev.html` 作为我们的 Playground。

#  createRoot & creatElement

写一个简单的例子找到 React 的入口：

```jsx
const App = () => {
  return <h1 onClick={() => console.log('click!')}>Hello World!</h1>
}
ReactDOM.createRoot(
  document.getElementById('container')
).render(
  <App />,
);
```

## createRoot

首先是 `createRoot` 函数，该函数位于 `react-dom/ReactDOMRoot.js`，主要用于：

1. 调用 `createContainer` 创建 `FiberRoot`
2. 调用 `listenToAllSupportedEvents` 为 `container` 绑定事件，React 的事件是通过 `container` 做事件代理来进行监听的
3. 调用 `new ReactDOMRoot` 创建 `ReactDOMRoot`，并返回这个对象，其原型链上有 `render` 和 `unmount` 方法

```typescript
export function createRoort(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  // ...
  // createContainer 根据 container 和初始信息创建一个 FiberRoot（FiberRoot 相关的详细内容见对应章节）
  const root = createContainer(container)
  container.__reactContainer$RandomKey = root
  // ...
  const rootContainerElement = container.nodeType === COMMENT_NODE
    ? container.parentNode
    : container;
  // listenToAllSupportedEvents 会在 contaienr 上添加 eventListenr 来监听支持的事件
  listenToAllSupportedEvents(rootContainerElement);
	// ReactDOMRoot.prototype 上定义了两个方法：render 和 unmount
  return new ReactDOMRoot(root);
}
```

调用 `createRoot` 返回的对象的 `render` 方法，他会调用 `updateContainer` 并传入一个由 jsx 转换来的 `ReactElement` 和刚刚创建好的 `FiberRoot`，来进入更新流程：

```typescript
ReactDOMRoot.prototype.render = function(children: ReactNodeList) {
  // ...
  updateContainer(children, root)
}
```

## createElement

Babel 会将 JSX 静态编译为 JavaScript 代码，可以在这个 [Babel Playground](https://babeljs.io/repl) 看到即时编译的效果，比如

```jsx
<div>123</div>
```

会被编译为：

```javascript
/*#__PURE__*/React.createElement("div", null, "123");
```

或者

```jsx
import { jsx as _jsx } from "react/jsx-runtime";
/*#__PURE__*/_jsx("div", {
  children: "123"
});
```

`jsx` 和 `createElement` 都定义在 `react/src/ReactElementValidator.js` 中了，他们都会在经过一系列校验之后调用 `react/src/ReactElement.js` 中对应的函数。

`jsx` 和 `createElement` 最终返回值都是相同的结构，这里以 `createElement` 举例：

```typescript
function createElement(type, config, children) {
  // ...
  // 对于函数组件，这里的 type 会传入这个函数本身
  // 从 config 中提取并生成 key, ref, props 等传给下一层
  return ReactElement(type, key, ref, soource, ReactCurrentOwner.current, props)
}
```

`ReactElement` 是一个十分简单的函数，他构造了一个 `element` 对象：

```typescript
function ReactElement(type, key, ref, self, source, owner, props) {
  const element = {
    // 标记该对象为 ReactElement 类型
    $$typeof: REACT_ELEMENT_TYPE,
    // 一些内置的属性
    type, key, ref, props,
    // 记录创建该元素的组件
    _owner: owner,
  }
  return element
}
```

最终，这个对象被传给了 `ReactDOMRoot` 的 `render` 函数，首屏渲染开始。
