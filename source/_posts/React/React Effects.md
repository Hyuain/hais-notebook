---
title: React Effects
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---
## Effects

- Effects 说的是不由某个特定的事件触发，而是由 **渲染本身** 触发的副作用。
- Effects 专门是指用来跟除了 React 以外的系统，比如浏览器 API、第三方组件、网络等进行同步。**如果我们只是基于某些 State 而改变另一些 State，可能不需要使用 Effect。**
- **Effects 在渲染之后执行**，这是一个同步 React 组件和外部系统的好时机。
- 由于闭包，每次执行 Effect 都有自己的 State。
- **每个 Effect 都应该描述了一个独立的同步事件**，不应该把不同的任务写在一个 Effect 中。
- React 通过 `Object.is` 来判断依赖是否发生了变化。

### useEffect

书写 Effect 需要三步：

1. 声明 Effect；
2. 确认依赖，什么时候执行该 Effect：
   1. 不加依赖表示每次渲染都执行，`[]` 表示首次渲染执行（mount）；
   2. 依赖应该只加那些 **Reactive** 值（响应式的），Props、States 以及 **其他所有在组件内部定义的数据** 都是 Reactive（因为他们是在渲染过程中计算出来的，并且属于 React 数据流的一员，是可能改变的）；
   3. `loaction.pathname` 这样的 Mutable 值不能被加入依赖。因为他可能在 React 数据流外部随意改变且不会触发重渲染。同时在渲染时读取 Mutable 数据也破坏了函数组件的纯粹性。最好使用 `useSyncExternalStore` 来订阅外部数据的变化；
   4. `ref.current` 这样的 Mutable 值也不能作为依赖。由 `useRef` 返回的 Ref 对象可以作为依赖，但是其 `current` 值是故意设置为 Mutable 的，他的变化不会触发重渲染。
3. 如果需要，增加清理函数。

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    // 1. 声明 Effect
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
    // 2. 确认依赖，首次渲染及 isPlaying 改变的时候执行
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}
```

如果在 Effect 中使用连接网络、开启计时器等时，可能需要在组件卸载（unmount）的时候进行清理，否则再次 mount 的时候可能会又开启好多重复的计时器。

**为了帮助及时发现问题，Strict Mode 下，React 会在开发环境中组件 mount 之后立即 remount 一下。**

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
  // 这个函数会在组件 unmount 的时候执行
  // 这个函数也会在下一次重渲染时 Effect 执行之前执行
  return () => {
    connection.disconnect();
  };
}, []);
```

### Data Fetching in Effects

在 Effects 中获取数据是一种非常常见的方法，但他也有一些缺陷：

- **在服务端渲染时无法使用。**浏览器需要下载所有的 JavaScript 代码再获取数据，比较低效。
- **容易产生“网络瀑布流”。**父组件获取数据，再渲染子组件，子组件再获取数据，串行结构会比较慢。
- **无法预加载或缓存数据。**
- **容易产生诸如竞态条件（Race Conditions）之类的 BUG。**

这其实是在 mount 的时候获取数据的库的通病，通常推荐以下方法解决：

- 使用框架（Next、Gatsby、Remix、Razzle）继承的数据获取机制；
- 使用或搭建一个客户端缓存，比如可以用 React Query、useSWR、React Router 6.4+ 等库。

以下是一个手动封装获取数据方法的简单例子：

```jsx
function SearchReasults({ query }) {
  const [page, setPage] = useState(1)
  const params = new URLSearchParams({ query, page })
  const results = useData(`/api/search?${params}`)
  
  function handleNextPageClick() {
    setPage(page + 1)
  }
}

function useData(url) {
  const [data, setData] = useState(null)
  useEffect(() => {
    let ignore = false
    fetch(url)
      .then(res => res.json())
      .then(json => {
        if (!ignore) {
          setData(json)
        }
      })
    return () => {
      ignore = true
    }
  }, [url])
  return data
}
```

#### Race Conditions

有时候会遇到这样的情况，组件接受一个 `id` 参数，当 `id` 改变的时候，就调用接口获取数据。此时，若接口返回的速度慢于 `id` 改变的速度，**屏幕上获取到的数据就会不停的变化**，最终停止为 **最后返回的数据**（不一定是最后一次 `id` 对应的数据）。

下面这段代码模拟了这种情况：

```jsx
  useEffect(() => {
    const fetchData = async () => {
      setTimeout(async () => {
        const response = await fetch(
          `https://swapi.dev/api/people/${props.id}/`
        );
        const newData = await response.json();
        setFetchedId(props.id);
        setData(newData);
      }, Math.round(Math.random() * 12000));
    };

    fetchData();
  }, [props.id]);
```

此时可以通过加 cleanup 函数的方式来解决。

第一种方法，是添加 `active` 或 `ignore` 标记来忽略掉之前几次的返回：

```jsx
useEffect(() => {
  let active = true;

  const fetchData = async () => {
    setTimeout(async () => {
      const response = await fetch(`https://swapi.dev/api/people/${props.id}/`);
      const newData = await response.json();
      if (active) {
        setFetchedId(props.id);
        setData(newData);
      }
    }, Math.round(Math.random() * 12000));
  };

  fetchData();
  return () => {
    active = false;
  };
}, [props.id]);
```

当 `props.id` 发生变化后，前面几次请求对应的 `active` 会被置为 `false`，并且他们的请求结果会被忽略。

第二种方法是直接中断前面的请求，比如通过 `abort`：

```jsx
useEffect(() => {
  const abortController = new AbortController();

  const fetchData = async () => {
    setTimeout(async () => {
      try {
        const response = await fetch(`https://swapi.dev/api/people/${id}/`, {
          signal: abortController.signal,
        });
        const newData = await response.json();

        setFetchedId(id);
        setData(newData);
      } catch (error) {
        if (error.name === 'AbortError') {
          // Aborting a fetch throws an error
          // So we can't update state afterwards
        }
        // Handle other request errors here
      }
    }, Math.round(Math.random() * 12000));
  };

  fetchData();
  return () => {
    abortController.abort();
  };
}, [id]);
```

### You Might Not Need an Effect

Effect 是用来跟外部系统交互的地方，如果我们不需要同步外部系统，通常是不需要 Effect 的。比如：

- **不需要通过 Effect 来转换数据。**比如想要 filter 一个列表，每次列表变动时我就让他自动执行 filter。然而，**这是低效的**。当我们更新组件的 State，React 会执行渲染函数并计算，然后 commit 到 DOM、更新视图，然后才会执行 Effects。如果 Effect 又要立即更新 State，整个过程都会重新执行一遍。**为了减少不必要的渲染过程，在组件最开头就转换数据，这些转换代码会在 State 改变的时候自动重新执行。**
- **不要用 Effects 来处理用户事件。**因为这样你就会找不到具体是哪里触发的事件了。

下面是一些具体的列子：

**更新一些基于 Props 或 State 的 State**

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

**缓存一些复杂的计算**

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 Avoid: redundant state and unnecessary Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ This is fine if getFilteredTodos() is not slow.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

```jsx
// `useMemo` 会让 React 记住其回调的返回结果，并且不会在 `todos` 和 `filter` 没有变化的时候重复执行。
// `useMemo` 包裹的函数将会在 **渲染（rendering）** 时执行，因此需要是纯函数。
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

**当 Props 改变的时候重置所有的 State**

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 Avoid: Resetting state on prop change in an Effect
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  const [comment, setComment] = useState('');
  // ...
}
```

**当某个 Prop 改变的时候改变 State**

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  // 因为当 React 遇到下面代码的时候就会在本次组件函数执行完成之后立即重新执行
  // 在这个时候，React 还没有渲染他的子组件，也没有更新 DOM
  // 为了减少级联，React 只允许在渲染时更新自己组件的 State，而不能更新其他组件的 State
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

**在事件处理函数中共享逻辑**

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  // Effect 只能用于“展示”本身就触发了事件，而不能用于本来就是有清晰事件触发者的情况
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

**发送一个 POST 请求**

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic runs because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

**计算链**

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

**应用初始化**

```jsx
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

```jsx
if (typeof window !== 'undefined') { // Check if we're running in the browser.
   // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

**通知父组件 State 变化**

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

```jsx
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

**给父组件传值**

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

**订阅外部数据**

```jsx
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```


