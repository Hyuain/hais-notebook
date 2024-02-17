---
title: useState
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

`useState` 是 React 提供的一个方便管理状态的 Hook，简单用法如下：

```jsx
const [n, setN] = useState(0)
```

可以查看 [CodeSandbox 上的这个例子](https://codesandbox.io/s/silly-diffie-9yk38)。

```jsx
const Grandson = () => {
  const [n, setN] = React.useState(0);
  const [m, setM] = React.useState(0);
  // 相当于
  // const array = React.useState(0)
  // const n = array[0]
  // const setN = array[1]
  // setN 会得到一个新的 n
  return (
    <div className="Grandson">
      孙子 n: {n}
      <button onClick={() => setN(n + 1)}>+1</button>
      m: {m}
      <button onClick={() => setM(m + 1)}>+1</button>
      {/* 函数组件的 setState 不会自动合并，建议分开写 */}
    </div>
  );
};
```

{% note warning %}

注意与 class 组件的 setState 不同，他是 **不能** 只更新对象的某个部分的；

并且，如果对象修改前后的地址不变，则不会触发重新渲染；

可以使用 `use-immer` 库来简单地写出不更改原有对象和数组的代码

{% endnote %}

# State as Snapshot

**每次渲染的 state 是固定的，调用 set 函数的时候，使用的还是此次渲染的值。**

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

点击 `+3` 按钮后，`number` 会变成 `1`，而不是 `3`，因为在此次渲染中，`number` 一直是 `0`，上面代码相当于这样：

```jsx
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

# Batching

为了减少无效的重复渲染，React 会在等待所有的事件处理函数结束，才会重渲染并更新 state。这被称为批处理（**batching**）。

React 并不会 batch 多个故意事件（比如点击事件，每个点击事件都会被单独处理）。比如第一次点击按钮的时候禁用了这个按钮，第二次点击就不会生效。

```jsx
setColor('orange');
setColor('pink');
setColor('blue');
```

只有 `setColor('blue')` 生效了。

# State Updating Queue

可以给 setter 传一个函数，这个函数被称为 **Updater Function**。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

这里给 setter 传了函数 `n => n + 1`：

- React 将此函数入队，等待事件处理函数中别的代码执行完成之后再执行；

- 在下次渲染调用 `useState` 时，React 遍历这个队列并且给出最终更新的 state。

有两种特殊情况（`number` 初始均为 0）：

```jsx
<button onClick={() => {
  setNumber(number + 5); // React 将“把值设置成0+5”入队
  setNumber(n => n + 1); // React 将函数入队
}}>
```

最终结果是 6。

```jsx
<button onClick={() => {
  setNumber(number + 5); // React 将“把值设置成 0 + 5” 入队
  setNumber(n => n + 1); // React 将函数入队
  setNumber(42); // React 将“把值设置成42”入队
}}>
```

最终结果是 42。

当事件处理函数完成，React 会触发重渲染。在重渲染中，React 会处理这个队列。所以 **updater functions 必须是纯函数**。不能在 updater functions 里面 setState 或执行其他副作用。

# Behind the Hook

> 为什么 useState 的时候没给名字也不会搞混呢？

以下面这段代码举例：

```jsx
 function RenderFunctionComponent() {
   const [firstName, setFirstName] = useState("Rudi");
   const [lastName, setLastName] = useState("Yardley");
 
   return (
     <Button onClick={() => setFirstName("Fred")}>Fred</Button>
   );
 }
```

首先会初始化两个空数组：`setters` 和 `state`，将 `cursor` 指向 `0`。

第一次渲染时，每次遇到 `useState`，会 push **一个 setter 函数** 进 setters 数组，和一个 **state** 进入 state 数组（这都基于 `cursor` 的位置）：

```
 cusor = 0
 
 const [firstName, setFirstName] = useState("Rudi");
 STATE = ["Rudi"]
 SETTERS = [setFirstName]
 cursor++
 
 const [lastName, setLastName] = useState("Yardley");
 STATE = ["Rudi", "Yardley"]
 SETTERS = [setFirstName, setLastName]
```

之后的每次渲染中，`cursor` 都将重新指向 `0`，并且依次从每个数组中读取数据。

如果 setter 被调用了，他就会将 state 中对应的某一项更新。

## myUseState

```jsx harmony
let _state = []
let index = 0
const myUseState = (initialValue) => {
  const currentIndex = index
  _state[currentIndex] = _state[currentIndex] === undefined ? initialValue : _state[currentIndex]
  const setState = (newValue) => {
    _state[currentIndex] = newValue
    render() // 在这里做一个简化
  }
  index ++
  return [_state[currentIndex], setState]
}

const render = () => {
  index = 0 // 这句话很关键，每次渲染之后 index 变成 0
  ReactDOM.render(<App/>, document.getElementById('root'));
}
```

由于是使用数组来实现 state，导致其对顺序依赖非常大，Hook 在每次渲染中必须以 **完全一样的顺序来调用**，比如 React 中不允许使用这种代码：

```jsx harmony
if (n % 2 === 1) {
  [m, setM] = React.useState(0)
}
```

每个函数组件对应一个 React 节点，每个节点将会保存 state（`memorizedState`） 和 index（链表）

## Preserving and Resetting State

> React 是如何保存函数组件的 State 的？

- 就像 DOM 和 CSSOM 一样，React 创建了一棵 UI 树。React 会根据这棵 UI 树去渲染 DOM。
- 每个组件都有独立的 State，但是这个 State 并不是存在于组件内部的，而是跟组件在 UI 树上的位置绑定的。
- 当 React 移除某个组件的时候，State 也随之销毁了。
- 同样的组件在同样的位置会有相同的 State（即使看起来是销毁了重建了一个）。
- 不同的组件会重置整个子树的 State。
  - 不要在组件里面定义组件，因为这样做的话，外层组件每次渲染都会创建一个 **不同** 的内层组件，导致内层组件状态丢失。
- 如果想要在组件移除之后仍然保持 State。
  - 一开始就全部渲染出来，通过 CSS 来控制是否显示：适合简单的 UI，渲染超级大的树会有性能问题；
  - 提升 State 到父组件去管理：最通用的方法；
  - 使用除了 State 以外的数据源：`localStorage` 等。

### Same component at the same position preserves state

**同样的组件在 UI 树上的同一个位置将会有相同的 State：**

```jsx
const App = () => {
  const [isFancy, setIsFancy] = useState(false)
  // Counter 里面的 State 将会被保留
  // 因为这里一直有一个 Counter 在同样的位置
  return (<>
  	{isFancy
      ? <Counter isFancy={true} />
      : <Counter isFancy={false} />}
  </>)
}
```

注意是在 **UI 树** 的同一个位置，并不是在 JSX 中的同一个位置（React 并不知道我们在哪里写条件判断，他只关心我们返回的这棵树）！

比如下面这种情况下，State 也是保留的（都是 `<Counter />` 在 `<div>` 的第一个子节点）：

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```

### Reset state at same position

有两个方法可以重置同样位置的 State。

方法一是将他们在不同位置渲染：

```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

`isPlayer` 为 `true` 时，第一个位置是 `Counter`，第二个位置为空；`isPlayer` 为 `false` 时，第一个位置为空，第二个位置是 `Counter`。

每个 `Counter` 的 State 都会在组件从 DOM 中移除的时候销毁。

**更加普遍是方法是使用 Key 来让 React 区分不同的组件**：

- 默认情况下，React 使用在父节点下的顺序来区分组件；
- Key 会显式地告诉 React 这是一个 *特别* 的组件。

```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```

注意 Keys 不是全局唯一的，他只是指明了组件在父结点下面的位置。