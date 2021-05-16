---
title: Next
date: 2021-04-07 22:39:22
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

NodeJS WEB 框架之 Next。
Tip：如何保存密码？
1. 存在环境变量中：用 `export xxx=` 设置环境变量，用 `process.env.xxx` 使用环境变量
2. 或者参考 [这篇文章](https://nextjs.org/docs/basic-features/environment-variables)

<!-- more -->

# 开始一个 Next.js 项目

```bash
npx create-next-app next-demo
```

# Next 的一些特性

## 使用 Link 快速导航

```jsx
// 原来这样写
<a href="xxx">点击跳转</a>
// 现在这样写
<Link href="xxx"><a>点击跳转</a></Link>
```

- 页面不会刷新，用 AJAX 请求新页面的内容
- 不会请求重复的 HTML、CSS、JS
- 自动在页面插入新内容、删除旧内容
- 省去了很多请求和解析的过程、速度极快

## 同构代码

同一份代码，在浏览器和 Node 两端都执行了，比如在组件里面写一个 console.log，会发现 Node 控制台和 Chrome 中都会有 log。

但：
- 不是所有的代码都会在两端运行，比如需要用户触发的操作，就不会在 Node 中运行
- 不是所有的 API 都能用，比如 `window` 就会报错，因为在 Node 中不存在

## `<Head>`

可以用 `<Head>` 标签书写 `title` `meta:viewport` 等。

```html
<Head>
  <title>My Blog</title>
  <link rel="icon" href="/favicon.ico" />
  <meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover" />
</Head>
```

也可以在 `pages/_app.js` 中进行全局配置：

```jsx
function MyApp({ Component, pageProps }) {
  return (
    <>
      <Head>
        <title>My Blog</title>
      </Head>
      <Component {...pageProps} />
    </>
  )
}

export default MyApp
```

切换页面的时候 MyApp 不会被销毁，MyApp 里面的代码会重新执行，但是里面的状态会保留。

## Style

在 `_app.js` 中还可以用 `import` 直接引入全局生效的 `global.css` 文件

```js
import '../styles/globals.css'
```

默认情况下都是相对路径引入，用配置指定根目录，可以参考 [这篇文档](https://nextjs.org/docs/advanced-features/module-path-aliases)

也可以在组件中书写局部的 `<style>`

```jsx
<style jsx>{`
  h1 { 
    color: red;
  }
`}</style>
```

## 静态资源

官方推荐静态资源放在 public 中，但在这里面的资源不能加 hash。 因此不推荐这样用，可以另外新建 `asstes/images` 文件夹来存放图片静态资源。
但直接这样就需要自己配置 file-loader，也就是自定义 webpack 配置：

```js
// next.config.js
module.exports = {
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.(png|jpg|jpeg|gif|svg)$/,
      use: [{
        loader: "file-loader",
        options: {
          name: "[name].[contenthash].[ext]",
          outputPath: "static",
          publicPath: "_next/static",
        }
      }]
    })
    return config
  }
}
```

也可以使用 [next-images 插件](https://github.com/twopluszero/next-images#readme) 来引入图片

# Next.js API 模式

在 api 目录下创建文件 `v1/post.tsx`

```tsx
import { NextApiHandler } from "next"
import p from "path"
import fs, { promises as fsPromise } from "fs"
import matter from "gray-matter"

export const getPosts = async () => {
  const markdownDir = p.join(process.cwd(), "markdown")
  const fileNames = await fsPromise.readdir(markdownDir)
  const posts = fileNames.map((name) => {
    const id = name.replace("/.md$/", "")
    const filePath = p.join(markdownDir, name)
    const text = fs.readFileSync(filePath, "utf-8")
    const { data: { title, date }, content } = matter(text)
    return {
      id, title, date,
    }
  })
  return posts
}

// 注意这里跟 express 和 koa 不同，没有将 next 直接暴露给我们，但是由于 next 基于 express，他其实也支持 express 类似的中间件，具体请查阅文档
const Posts: NextApiHandler = async (request, response) => {
  const posts = await getPosts()
  response.statusCode = 200
  response.setHeader("Content-Type", "application/json")
  response.write(JSON.stringify(posts))
  response.end()
}

export default Posts
```

可以试着访问 `/api/v1/posts`，这段代码只会运行在 node，不会运行在浏览器中

# Next.js 三种渲染方式

## 客户端渲染（BSR）

在浏览器上执行的渲染（Browser Side Render），类似用 JS（Vue、React）去创建 HTML

缺点：
- 白屏，而且在拿到响应之前都需要显示 loading 状态
- SEO 不友好，搜索引擎拿不到具体的数据；搜索引擎也不会执行 JS，因此看到的 HTML 信息量极少

```tsx
// pages/posts/index.tsx
import { NextPage } from "next"
import axios from "axios"
import { useCallback, useEffect, useState } from "react"

type Post = {
  id: string
  date: string
  title: string
}

const PostsIndex: NextPage = () => {
  const [posts, setPosts] = useState<Post[]>([])

  useEffect(() => {
    axios.get("/api/v1/posts").then((res) => {
      setPosts(res.data)
    })
  }, [])

  const handleClick = useCallback(() => {
    console.log("clicked")
  }, [])

  return (
    <div>
      {/* 静态内容 */}
      <h1 onClick={handleClick}>文章列表</h1>
      {/* 动态内容 */}
      {
        posts.map((item) => <div key={item.id}>{item.title}</div>)
      }
    </div>
  )
}

export default PostsIndex
```

实际上上述的静态内容是先由服务器渲染出来，然后再在浏览器上增加事件绑定的，可以看 [这篇文档](https://zh-hans.reactjs.org/docs/react-dom-server.html#rendertostring)：

服务端调用 `ReactDOMServer.renderToString(element)`，将 React 元素渲染为初始 HTML。
再在前端调用 `ReactDOM.hydrate()`，会在标记过的节点上绑定事件。

但事实上，前端也会渲染一次，检查一下前端跟服务端渲染的内容是否一致（前端渲染的目的是为了检查一致性，而不是为了替换）。
如果我们在函数组件最开始打一个 debugger，然后趁 debugger 的时间在页面上把服务端渲染出来的 HTML 改一下，再继续执行，会发现报错：

![如果前后端渲染不一致，会报错](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-1.png)

## 静态页面生成（SSG）

静态页面生成（Static Site Generation）是预渲染（Pre-rendering）的一种，类似于把 PHP 提前渲染成 HTML。

时机：
  - 生产环境中，静态化是在 `yarn build` 的时候实现的
优点：
  - 能解决白屏问题
  - 解决 SEO 问题
  - 前端重复渲染相同内容的问题（可以将动态内容静态化，将需要客户端渲染的动态内容提前由服务端渲染好，然后作为静态内容发给前端）
缺点： 
  - 无法生成用户相关内容（每个人看到的都是一样的）

```tsx
import { GetStaticProps, NextPage } from "next"
import axios from "axios"
import { useCallback, useEffect, useState } from "react"
import { getPosts } from "../../lib/posts"

type Post = {
  id: string
  date: string
  title: string
}

type Props = {
  posts: Post[]
}

const PostsIndex: NextPage<Props> = (props) => {
  // 这个 props 其实前后端都能拿到
  const { posts } = props
  return (
    <div>
      <h1>文章列表</h1>
      {
        posts.map((item) => <div key={item.id}>{item.title}</div>)
      }
    </div>
  )
}

// 注意这里不要忘了 export，如果不写的话，这段代码会被前端执行，会报错
// 在生产环境中，是在 build 的时候执行的
export const getStaticProps: GetStaticProps = async () => {
  // 服务端显然没必要通过 AJAX 去拿数据，直接去文件系统或者数据库拿
  const posts = await getPosts()
  return {
    props: {
      posts: JSON.parse(JSON.stringify(posts)),
    }
  }
}

export default PostsIndex
```

前端也能拿到 props，因为 Next 会将这些数据放在 HTML 中给前端：

![Next 会将数据放在 HTML 中给前端](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-2.png)

### 同构代码的好处

将数据放在 HTML 中之后，前端也能很方便地使用了，其实其他后端语言也能做到，但 Next.js 更加方便，因为：

- Next 支持 jsx，写出来的代码可以与 React 无缝对接，可以只写一份代码
- Next 写出来的对象都是 JavaScript 对象，无需类型转换，如果是其他语言就需要进行额外的类型转换

### yarn build

`yarn build` 的时候会提示你页面的类型：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-3.png)

比如空心圆代表静态页面，而实心圆代表使用了 getStaticProps 进行静态化。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-3.png)

可以看到编译之后，一个 SSG 页面（实心圆）变成了三个文件：

- html 是用户拿到的静态页面
- js 里面也含有静态内容，是快速导航（Link）需要用到的，因为快速导航实际上是使用 js + json 做的，而不是再去请求一次 html
- json 是 getStaticProps 得到的数据，为了方便 js 接入不同的数据，因此 js 和 json 需要分开

### 根据路径创建静态页面

```tsx
// pages/posts/[id].tsx
import { getPost, getPostIds } from "../../lib/posts"
import { GetStaticPaths, GetStaticProps, NextPage } from "next"
type Props = {
  post: Post
}

type Params = {
  id: string
}

const PostShow: NextPage<Props> = (props) => {
  const { post } = props
  return (
    <div>
      <h1>{post.title}</h1>
      <article>
        {post.content}
      </article>
    </div>
  )
}

export default PostShow

// 声明路由：用这个来穷举静态页面的不同路径
export const getStaticPaths: GetStaticPaths = async () => {
  const idList = await getPostIds()
  const paths = idList.map((id) => {
    return {
      params: { id },
    }
  })
  return {
    paths,
    // false：当请求的 id 不再 path 里面，直接返回 404
    // true：若不存在，则依然渲染页面
    fallback: false,
  }
}

// 通过 context 中的 params 拿到上一个函数穷举出的路径，也就是我们的 id，然后借由 id 去获取内容并生成静态文件
export const getStaticProps: GetStaticProps<Props, Params> = async (context) => {
  const id = context.params!.id
  const post = await getPost(id)
  return {
    props: {
      post,
    }
  }
}
```

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/next-5.png)

可以看到生成来的文件中只有一个 JS，但是有两个 JSON 和两个 HTML 文件，这就是根据不同路由的到的不同页面和数据。

## 服务端渲染（SSR）

服务端渲染（Server Side Render）也是预渲染的一种，类似于 PHP、Python、Ruby、Java 后台的基本功能。
比如当需要通过不同的用户 id 等渲染不同的内容，这时候就较难提前做静态化，这个时候就可以使用 SSR，由服务端渲染好首屏，再下拉刷新。

时机：
  - 无论是在开发环境还是生产环境，都是每次请求来到之后运行的
优点：
  - 能解决白屏问题
  - 解决 SEO 问题
  - 可以生成跟用户有关系的内容
缺点：
  - 无法获取客户端信息，比如浏览器窗口大小等

```tsx
import { GetServerSideProps, NextPage } from "next"
import { IBrowser, UAParser } from "ua-parser-js"
import { useEffect, useState } from "react"

type Props = {
  browser: IBrowser
}

const Home: NextPage<Props> = (props) => {
  const { browser } = props
  const [clientWidth, setClientWidth] = useState(0)
  // 如果想要获取浏览器窗口的大小，不能使用 SSR，只能使用客户端渲染，这样的写法保证了下面的代码只会在浏览器中运行，防止 node 报错
  useEffect(() => {
    setClientWidth(document.documentElement.clientWidth)
  }, [])
  return (
    <>
      <h1>你的浏览器是 {browser.name}</h1>
      <h1>你的窗口宽度是 {clientWidth}</h1>
    </>
  )
}

export default Home

// 每次请求的时候才会执行
export const getServerSideProps: GetServerSideProps = async (context) => {
  // context 中有 req 和 res，一般只需要 req
  const ua = context.req.headers["user-agent"]
  const result = new UAParser(ua).getResult()
  return {
    props: {
      browser: result.browser,
    }
  }
}
```

## 三种渲染应该如何选择

1. 有动态内容吗？
  - 有，跳到 2
  - 没有，直接写 HTML 即可
2. 跟客户端相关（比如获取窗口大小等）吗？
  - 相关，BSR
  - 无关，跳到 3
3. 我的响应跟用户或请求相关吗？
  - 相关，SSR（getServerSideProps）或 BSR
  - 无关，SSG（getStaticProps） 或 SSR 或 BSR