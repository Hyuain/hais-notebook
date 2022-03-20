---
title: Redux
date: 2020-02-04 09:50:36
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

主要记录了 Redux 的相关东西。

可以看看 [这个例子](https://codesandbox.io/s/jovial-gates-wbs8e)：

```js
import { createStore } from "redux";

const reducer = function(previousState, action) {
  if (previousState === undefined) {
    previousState = 0;
  }
  switch (action.type) {
    case "INCREMENT":
      return previousState + 1;
    case "DECREMENT":
      return previousState - 1;
    default:
      return previousState;
  }
};

const store = createStore(reducer);

const render = function() {
  document.getElementById("value").innerHTML = store.getState();
};

render();

store.subscribe(render); // 每次 dispatch 就会触发 render

const addOne = function() {
  store.dispatch({ type: "INCREMENT" });
};

const minusOne = function() {
  store.dispatch({ type: "DECREMENT" });
};

document.getElementById("increment").addEventListener("click", addOne);
document.getElementById("decrement").addEventListener("click", minusOne);
document.getElementById("incrementIfOdd").addEventListener("click", () => {
  if (store.getState() % 2 === 1) {
    addOne();
  }
});
document.getElementById("incrementAsync").addEventListener("click", () => {
  setTimeout(addOne, 1000);
});
```

