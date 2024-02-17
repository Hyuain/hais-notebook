---
title: ReactDOMRoot
date: 2023-06-23 16:41:41
categories:
  - - 前端
tags:
  - FX
---

ReactDOMRoot 是 `ReactDOM.createRoot` 的返回值，他事实上只是暴露了两个方法给我们使用，一个是 `render`，一个是 `unmount`。我们可以给 `render` 方法传入 `ReactElement`，然后开启首次渲染。