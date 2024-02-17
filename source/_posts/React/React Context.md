---
title: React Context
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

Context 类似于 Vue / Angular 的 Provide / Eject，使得数据可以跨层传递，而不必显式地通过组件树逐层传递 props。

像 CSS 属性继承，下面的组件都会得到这个 Context，只能用过上层新的 Context Provider 才能覆盖更上层传下来的值。不同的 Context 不会相互覆盖。

**只是需要将属性传递很多层并不意味着需要使用 Context**，在使用 Context 之前还需要考虑一些替代方法：

- **使用 Props**。这样会比较清晰哪个组件使用了什么数据。

- **提取组件，然后把 JSX 作为 children 传给他们**。这样可以越过中间不需要知道这个数据的层。

  ```jsx
  <Layout posts={posts} />
  // 改为
  <Layout><Posts posts={posts} /></Layout>
  ```

Context 的典型使用场景：主题、账号、路由、状态管理。

第一个例子：

```jsx
// 创建一个 context
const C = createContext(null)

const App = () => {
  const [n, setN] = useState(0)
  return (
    // 圈定作用域并给初始值（初始值通常是一个读接口和写接口）
    <C.Provider value={{n, setN}}>
      <Child/>
    </C.Provider>
  )
}

const Child = () => {
  // 子组件拿到读接口和写接口
  const {n, setN} = useContext(C)
  return (
    // ...
  )
}
```

第二个例子：

```jsx
const LevelContext = createContext(0)

const Page = () => {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );

const Section = ({ children }) => {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

const Heading = ({ children }) => {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```
