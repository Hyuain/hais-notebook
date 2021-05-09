---
title: Webpack 入门尝试
date: 2020-02-02 15:37:40
tags:
  - 入门
  - 效率
  - 收集
  - 踩坑
categories:
  - [工具, Webpack]
---

webpack 的基础使用及问题。

<!-- more -->

{% note warning %}
webpack 安装的所有东西都是 --dev
{% endnote %}

# Loader 与 Plugin

- 翻译
- 解释：加载文件；拓展功能
- 加载一个个 JS 文件，把 JS 文件转换成低版本浏览器可以支持的（JS Loader）；加载 CSS 文件，把 CSS 变成标签，或者进行其他处理 （CSS Loader、Style Loader）；加载图片，对图片进行优化；
- HTML Webpack Plugin 生成 HTML 文件
- Mini CSS Extract Plugin 抽取 CSS 文件

# 加载 JavaScript 文件

可以这样来调用本地安装的 webpack：

```bash
./node_modules/.bin/webpack —version 
# 或者用 npx，但 npx 不够稳定，比如如果 node 装到了有空格的目录，可能就有问题
npx webpack
```

webpack 自带了 JS Loader，我们需要自定义一下 `webpack.config.js`

```js
var path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.js', // 默认为 './src/index.js'
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js'
  }
}
```

> **HTTP Cache**
> Response Header: Cache-Control

可以设置 `package.json` 中的 `script`，让他每次自动删除之前的 `dist` 目录

```json
"scripts": {
  "build": "rm -rf dist && webpack"
}
```

之后只要每次用：

```bash
npm run build
# 或者
yarn build
```

# 生成 HTML 文件

可以使用 [html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/)

首先进行安装

```bash
npm install --save-dev html-webpack-plugin
```

使用 html-webpack-plugin 需要修改 `webpack.config.js`

```js
var HtmlWebPackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebPackPlugin({
      title: 'my title',
      template: 'src/assets/index.html'
    })
  ]
}
```

此外，在 template 文件里面的 title 需要这样写：

```html
<title><%= htmlWebpackPlugin.options.title %></title>
```

# 加载 CSS 文件

可以使用 [css-loader](https://webpack.js.org/loaders/css-loader/) 和 [style-loader](https://webpack.js.org/loaders/style-loader/)

首先进行安装

```bash
npm install --save-dev css-loader style-loader
```

使用 css-loader 和 style-loader 需要修改 `webpack.config.js`

```js
module: {
  rules: [
    {
      test: /\.css$/i,
      use: ['style-loader', 'css-loader'],
    }
  ]
}
```

{% note warning %}
CSSLoader 会把 .css 后缀的文件读到 JS 里面
StyleLoader 会把 style 标签放到 HTML 的 head 里面
{% endnote %}

# 预览与调试网页

可以使用 [webpack-dev-server](https://webpack.js.org/guides/development/)

首先进行安装

```bash
npm install --save-dev webpack-dev-server
```

使用 webpack-dev-server 同样需要修改 `webpack.config.js`

```js
devServer: {
  contentBase: './dist'
}
```

然后再在 `package.json` 里面加入 `script` 方便使用

```json
"scripts": {
  "start": "webpack-dev-server --open",
}
```

# 提取 CSS

可以使用 [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)

首先进行安装

```bash
npm install --save-dev mini-css-extract-plugin
```

使用 mini-css-extract-plugin 需要修改 `webpack.config.js`

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // all options are optional
      filename: devMode ? '[name].css' : '[name].[hash].css',
      chunkFilename: devMode ? '[id].css' : '[id].[hash].css',
      ignoreOrder: false, // Enable to remove warnings about conflicting order
    }),
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              // you can specify a publicPath here
              // by default it uses publicPath in webpackOptions.output
              publicPath: '../',
              hmr: process.env.NODE_ENV === 'development',
            },
          },
          'css-loader',
        ],
      },
    ],
  },
};
```

# 加载 Sass

{% note warning %}
node sass 已经过时，应该使用 dart sass
{% endnote %}

可以使用 [sass-loader](https://webpack.js.org/loaders/sass-loader/)

同样需要修改 `webpack.config.js`

```js
module: {
    rules: [
      {
        test: /\.scss$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'sass-loader',
            options:{implementation: require('dart-sass')}
          },
        ],
      },
    ]
  },
```

# 加载 Less

可以使用 [less-loader](https://webpack.js.org/loaders/less-loader/)

# 加载 Stylus

可以使用 [stylus-loader](https://github.com/shama/stylus-loader)

# 加载图片

可以使用 file-loader，把文件变成文件路径

# 懒加载

等到真正需要的时候再加载

```js
const promise = import('./lazy.js')
// 将会得到一个 Promise 对象
promise.then((module) => {
	module.default() // 将会执行 export default 导出的函数
}, () => {
})
```

# 在 GitHub Pages 上部署

第一次

```bash
git branch gh-pages
git checkout gh-pages
# 删除别的，只留下 dist、node_modules 和 .gitignore
mv dist/* ./
rm -rf dist
```

以后

```bash
npm run build: &&
git checkout gh-pages &&
rm -rf *.html *.js *.css *.png &&
mv ./dist/* ./ &&
rm -rf dist
git add . &&
git commit -m 'update' &&
git push &&
git checkout -
```