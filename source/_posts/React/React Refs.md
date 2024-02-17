---
title: React Refs
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---
# Referencing Values

```jsx
const ref = useRef(0)
// 之后通过 ref.current 访问及修改
```

Refs 的修改不会触发组件的重新渲染。可以用 Ref 存储计时器 ID、DOM 元素等不希望改变组件渲染结果（JSX）的东西。

**不要在渲染中访问及修改 Ref**（应该在事件处理函数中使用），因为 Ref 的更新并不会触发重新渲染，使得渲染中使用的 Ref 仍然是旧值。

比如下面代码就是一个不会被更新的 Ref：

```jsx
<button onClick={handleClick}>
  You clicked {countRef.current} times
</button>
```

# Refs and DOM

```jsx
export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

## Ref Callback

可以给 Ref 传一个回调函数，React 会在设置 Ref 的时候调用这个回调函数，并传递一个 DOM 节点作为参数；在清除的时候也会调用函数并传递 `null` 作为参数。

这样就可以管理一堆 Ref 列表了。

```jsx
export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```

# forwardRef

 `forwardRef` 允许组件接收 ref 并将其向下传递，最终可以使得 ref 指向最底层的 HTML DOM

```jsx harmony
const MyComp = () => {
  // const myRef = React.createRef()
  const myRef = useRef(null)
  
  return (
    <Child ref={myRef}> I am a Button </Child>
  ) imperative
}
const Child = React.forwardRef((props, ref) => (
  <button ref={ref} className="button-wrapper">
    {props.children}
  </button>
))
```

通过这样的方式，我们在 MyComp 中就可以通过 `myRef.current` 获取到原生的 button 了

## Imperative Handle

`useImperativeHandle` 让我们可以自定义父组件访问到的子组件 Ref 的内容：

```jsx
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}

```

## When React Attahces the Refs

如前面所说，一次更新分为渲染（Render）和提交（Commit）两个阶段。

- 渲染过程中一般不访问 Ref（这一点在引用值的时候已经说明了）：
  - 在首次渲染时，DOM 节点没有被创建，因此 `ref.current` 是 `null`；
  - 重新渲染时，DOM 节点并没有被更新，因此获取 Ref 也太早了。
- React 是在提交时设置 `ref.current` 的：
  - 更新 DOM 之前，React 会将受到影响的 `ref.current` 设置为 `null`；
  - 更新 DOM 之后，React 立即将他们设置成对应的 DOM 节点。
- 通常需要在事件处理函数中访问 Refs。
- 如果没有合适的事件处理函数的话，也可以在 Effect 中访问 Refs。

## flushSync

下面代码中，我们试图增加一个新的 TODO 后，立即滚动到新增的那一项：

```jsx
export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

但是这是做不到的，因为 `setTodos` 并不会立即更新 DOM，因此我们滚动的时候，还拿不到最新的那个节点。

可以通过 `flushSync` 来让 React 在其包裹的代码执行完成之后，**同步地** 更新 DOM：

```jsx
flushSync(() => {
  setTodos([ ...todos, newTodo ])
})
listRef.current.lastChild.scrollIntoView()
```

这样就可以拿到最新的 DOM 了。
