---
title: axios
date: 2020-02-02 17:36:25
categories:
  - [工具]
---

文档翻译
用于浏览器和 node.js 的基于 Promise 的 HTTP 客户端。

<!-- more -->

# 特性

- 从浏览器发起 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- 从 node.js 发起 [http](http://nodejs.org/api/http.html) 请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 拦截请求和响应
- 转换请求和响应数据
- 取消请求
- JSON 数据的自动转换
- 从客户端防范 XSRF 攻击

# 安装

使用 npm：

```bash
$ npm install axios
```

使用 bower：

```bash
$ bower install axios
```

使用 yarn：

```bash
$ yarn add axios
```

使用 cdn：

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

# 例子

## 注意：这是在 CommonJS 中的使用方法

为了在使用 `require()` 进行导入的 CommonJS 中获得 TypeScript 输入时的智能补充和类型提示，需要使用这样的方法：

```js
const axios = require('axios').default;
// 这样 axios.<method> 就可以提供自动补充和参数类型了
```

执行一个 `GET` 请求：

```js
const axios = require('axios');

// 在已知 ID 的情况下请求 user
axios.get('/user?ID=12345')
  .then(function (response) {
  // 处理成功的情形
  console.log(response);
  })
  .catch(function (error) {
  // 处理失败的情形
  console.log(error);
  })
  .then(function () {
  // 永远执行
  });

// 上面的请求也可以这样写：
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
  // 处理成功的情形
  console.log(response);
  })
  .catch(function (error) {
  // 处理失败的情形
  console.log(error);
  })
  .then(function () {
  // 永远执行
  });

// 想要使用 async/await？那就在你外面的函数/方法上加上 `async` 关键字
async function getUser() {
  try {
    const response = await aixos.get('/user?ID=12345');
    console.log(response);
  } catch (error) {
    console.log(error);
  }
}
```

> **注意：**`async/await` 是 ECMAScript 2017 的一部分，并且不支持 Internet Explorer 和一些更久的浏览器，所以使用的时候要注意。

执行一个 `POST` 请求：

```js
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
```

执行多个并发请求：

```js
function getUserAccount() {
  return axios.get('/user/12345');
}
function getUserPermission() {
  return axios.get('/user/12345/permission');
}
axios.all([getUserAccount(), getUserPermission()])
  .then(axios.spread(function (acct,perms) {
    // 两个请求都完成了
  }));
```

# axios API

可以通过给 `axios` 传递相关的 config 来执行请求。

axios(config)

```js
// 发送一个 POST 请求
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
```

```js
// 使用 GET 方法请求一个图片
axios({
  method: 'get',
  url: 'http://bit.ly/2mTM3nY',
  responseType: 'stream'
})
  .then(function (response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
  });
```

axios(url[, config])

```js
// 发送一个 GET 请求（默认方法）
axios('/user/12345');
```

## 请求方法的别名

为了方便，所有支持的请求方法都提供了别名。

axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])

> 注意
> 当使用别名时，不需要再在 config 中指定 `url` `method` 和 `data` 属性

## 并发性

有一些帮助函数来处理并发请求。

axios.all(iterable)
axios.spread(callback)

## 创建一个实例

你可以使用自定义的 config 来创建一个 axios 实例。

axios.create([config])

```js
const instance = axios.create({
  baseURL: 'https://some-domain.com/api',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
})
```

## 实例方法

下面列举了可用的实例方法，指定的 config 会与实例的 config 合并。

axios#request(config)
axios#get(url[, config])
axios#delete(url[, config])
axios#head(url[, config])
axios#options(url[, config])
axios#post(url[, data[, config]])
axios#put(url[, data[, config]])
axios#patch(url[, data[, config]])
axios#getUri([config])

# 请求的 config

在众多的 config 选项中，只有 `url` 是必须的。如果没有指定 `method`，则默认为 `GET`。

```js
{
  // `url` 是被用于请求的服务器的 URL
  url: '/user',

  // `method` 是发起请求时用的方法
  method: 'get', // 默认值
  
  // `baseURL` 将在 `url` 不是绝对路径的时候放在 `url` 的前面
  // 给 axios 实例设置一个 `baseURL` 就可以很方便地给实例的方法传递相对路径
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在请求发送给服务器之前变换数据
  // 这只适用于 'PUT' 'POST' 'PATCH' 和 'DELETE' 方法
  // 数组中最后的函数必须返回一个字符串
  // 或者 Buffer、ArrayBuffer、FormData 或 Stream 的实例
  // 你可以修改 headers 对象
  transformRequest: [function (data, headers) {
    // 做些什么来修改数据
    return data
  }],

  // `transformResponse` 允许对响应数据在传递给 then/catch 之前进行变换
  transformResponse: [function (data) {
    // 做些什么来修改数据
    return data    
  }],
  
  // `headers` 是自定义的请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},
  
  // `params` 是请求时的 URL 参数
  // 必须要是一个纯粹的对象或者 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `paramsSerializer` 是一个可选的函数，用来控制序列化的 `params`
  // （比如 https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/）
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是被当做请求体发送的数据
  // 只适用于 'PUT' 'POST' 和 'PATCH' 方法
  // 当没有设置 `transformRequest` 时，必须是以下几种类型之一：
  // - 字符串、纯粹的对象、ArrayBuffer、ArrayBufferView、URLSearchParams
  // - 只有浏览器可用：FormData、文件、Blob
  // - 只有 Node 可用： Stream、Buffer
  data: {
    firstName: 'Fred'
  },
  
  // 发送请求数据的另一种写法
  // post 方法
  // 只有 value 被发送，key 不会
  data: 'Country=Brasil&City=Belo Horizonte',

  // `timeout` 指定了请求超时的毫秒数
  // 如果请求超过了 `timeout`，则请求将会被放弃
  timeout: 1000, // 默认为 `0` （没有时限）
  
  // `withCredentials` 指定了访问跨站点的 Access-Control 请求
  // 是否使用证书
  withCredentials: false, // 默认
  
  // `adapter` 允许自定义让测试变简单的请求控制器
  // 返回一个 promise 并且提供一个合法的响应
  adapter: function (config) {
    /* ... */
  },

  // `auth` 说明需要使用 HTTP 基本身份验证，并且提供合法的证书
  // 他将会 设置一个 `Authorization` 请求头
  // 并且会覆盖在 `headers` 中自定义的 `Authorization` 请求头
  // 请注意这里只能设置 HTTP 基本身份验证
  // 对于不记名令牌等，需要使用自定义的 `Authorization` 请求头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },
  
  // `responseType` 指定了服务器返回的数据类型
  // 包括 'arraybuffer' 'document' 'json' 'text' 'stream'
  // 只有浏览器可用：'blob'
  responseType: 'json', // 默认

  // `responseEncoding` 指定了解码响应数据的编码方式
  // 注意：在 'stream' 或客户端请求的 `responseType` 被忽略
  responseEncoding: 'utf-8', // 默认
  
  // `xsrfCookieName` 是用来做为 xsrf 令牌值的 cookie 的名字
  xsrfCookieName: 'XSRF-TOKEN', // 默认
  
  // `xsrfHeaderName` 是保存 xsrf 令牌值的 HTTP 头的名字
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认
  
  // `onUploadProgress` 允许控制上传的事件
  onUploadProgress: function (progressEvent) {
    // 对原生事件做些什么
  },
  
  // `onDownloadProgress` 允许控制下载的事件
  onUploadProgress: function (progressEvent) {
    // 对原生事件做些什么
  },

  // `maxContentLength` 定义了以字节计量的 HTTP 响应允许的最大容量
  maxContentLength: 2000,

  // `validateStatus` 定义根据 HTTP 响应的状态码
  // 是 resolve 还是 reject 这个 promise
  // 如果 `validateStatus` 返回 `true`（或设为 `null` 或 `undefined`），
  // 将会 resolve 这个 promise
  // 反之则 reject 这个 promise
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认
  }

  // `maxRedirects` 定义了在 node.js 中最大的重定向数
  // 如果设为 0，则不允许重定向
  maxRedirect: 5, // 默认
  
  // `socketPath` 定义了在 node.js 中使用的 UNIX Socket
  // 比如 '/var/run/docker.sock' 来发送请求到 docker daemon
  // `socketPath` 与 `proxy` 中只有一个生效
  // 如果都被设置了，则使用 `socketPath`
  socketPath: null, // 默认

  // `httpAgent` 和 `httpsAgent` 分别定义了一个执行 http 和 https 请求时
  // 在 node.js 中的代理
  // 这允许诸如 `keepAlive` 之类的默认情况下不允许的选项
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义了代理服务器的主机名和端口
  // 你也可以用常规的 `http_proxy` `https_proxy` 环境变量定义你的代理
  // 如果你正在使用环境变量来配置代理，你也可以定义 `no_proxy` 环境变量
  // 这是一个用逗号分隔的不应该被代理的主机列表
  // 用 `false` 来禁用代理，忽略环境变量
  // `auth` 表示在连接这个代理的时候应该使用 HTTP 基本身份验证，并提供证书
  // 这将会设置 `Proxy-Authorization` 请求头
  // 并且覆盖已经存在的在 `headers` 中自定义的 `Proxy-Authorization`
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定了一个可以被用来取消请求的取消令牌
  // （查看下面的取消部分来获取更多信息）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

# 响应模式

请求的响应包含了以下信息：

```js
{
  // `data` 是服务器提供的响应
  data: {},
  
  // `status` 是服务器返回的 HTTP 状态码
  status: 200,

  // `statusText` 是服务器返回的 HTTP 状态信息
  statusText: 'OK',
  
  // `headers` 是服务器返回的响应头
  // 所有的返回头都是小写
  headers: {},

  // `config` 是提供给 `axios` 做请求的 config
  config: {},
  
  // `request` 是产生这个响应的请求
  // 这是 node.js 中（在重定向中）的最后一个 ClientRequest 实例
  // 和浏览器中的 XMLHttpRequest 实例
  request: {}
}
```

当使用 `then` 的时候，你会收到这样的响应：

```js
axios.get('/user/12345')
  .then(function (response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.header);
    console.log(response.config);
  })
```

当使用 `catch` ，或者传递一个 rejection 回调作为 `then` 的第二个参数的时候，响应将在 `error` 对象中可用，就像在处理错误章节中说的一样。

# 配置的默认值

你可以指定一个默认配置应用于所有的请求。

## 全局 axios 默认配置

```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.helpers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

## 自定义实例默认配置

```js
// 在创建实例的时候设置默认配置
const instance = axios.create({
  baseURL: 'https://api.example.com'
});

// 在创建实例之后改变默认配置
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

## 配置的优先顺序

配置会根据优先顺序自动合并。首先是在 lib/defaults.js 里面的库默认配置，然后是实例中的 `defaults` 属性，最后是请求的 `config` 参数。后者的优先级比前者高。这里是一个例子：

```js
// 用库中提供的默认配置来创建一个实例
// 就这个例子来说，timeout 配置是 `0`，因为是库的默认值
const instance = axios.create();

// 覆盖库中默认的 timeout 值
// 现在这个实例的所有请求会在超时之前等待 2.5 秒
instance.defaults.timeout = 2500;

// 覆盖这个请求的 timeout 值，因为我们知道他会花更长的时间
instance.get('/longRequest', {
  timeout: 5000
});
```

# 拦截器

你可以在请求或者响应被 `then` 或 `catch` 处理之前拦截它们。

```js
// 添加一个请求拦截器
axios.interceptors.request.use(function (config) {
  // 在请求发出之前做些什么
  return config;
}, function (error) {
  // 在请求失败的时候做些什么
  return Promise.reject(error)
});

// 添加一个响应拦截器
axios.interceptors.response.use(function (response) {
  // 所有状态码在 2xx 范围的都会触发
  // 对返回的数据做些什么
  return response;
}, function (error) {
  // 所有状态码不在 2xx 范围的都会触发
  // 对返回的错误做些什么
  return Promise.reject(error)
})
```

如果你需要移除一个拦截器：

```js
const myInterceptor = axios.interseptors.request.use(function () {/*...*/});
axios.interseptors.request.eject(myInterceptor);
```

你可以给一个 axios 实例增加拦截器：

```js
const instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

# 处理错误

```js
axios.get('/user/12345')
  .catch(function (error) {
    if(error.response) {
      // 发出请求后收到一个状态码不在 2xx 范围的回复
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    } else if (error.request) {
      // 请求发出了但是没有收到回复
      // `error.request` 是浏览器中 XMLHttpRequest 的实例
      // 也是一个 node.js 中 http.ClientRequest 的实例
      console.log(error.request);
    } else {
      // 在引发错误的请求的安装过程中发生了些状况
      console.log('Error', error.message);
    }
    console.log(error.config);
  })
```

使用 `validateStatus` 配置选项，可以配置应该抛出错误的 HTTP 状态码。

```js
axios.get('/user/12345', {
  validateStatus: function (status) {
    return status < 500; // 只 reject 状态码大于等于 500 的响应
  }
})
```

使用 `toJSON` 可以获得关于 HTTP 错误的更多信息。

```js
axios.get('/user/12345')
  .catch(function (error) {
    console.log(error.toJSON());
  })
```

# 取消

你可以使用取消令牌来取消请求。

> axios 取消令牌 API 基于被弃用的 [cancelable promises proposal](https://github.com/tc39/proposal-cancelable-promises)

你可以用 `CancelToken.source` 工厂函数创建一个取消令牌：

```js
const CancelToken = axios.CancelToken();
const source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function (thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

axios.post('/user/12345', {
  name: 'new name'
}, {
  cancelToken: source.token
})

// 取消请求（消息参数是可选的）
source.cancel('Operation canceled by user.');
```

你也可以通过传递一个执行函数给 `CancelToken` 构造器来创建一个取消令牌：

```js
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // 一个执行函数接受取消函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

> 注意：你可以用同一个取消令牌取消多个请求。

# 使用 application/x-www-form-urlencoded 表格

默认情况下，axios 会将 JavaScript 对象序列化为 `JSON`。如果需要以 `application/x-www-form-urlencoded` 的格式来发送数据，你可以下面的选项之一：

## 浏览器

在浏览器中，你可以使用 [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) API：

```js
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);
```

> 注意并不是所有的浏览器都支持 `URLSearchParams`，但是这里有一个可用的 [polyfill](https://github.com/WebReflection/url-search-params)（务必确认 polyfill 全局环境）

你也可以用 [`qs`](https://github.com/ljharb/qs) 库进行编码：

```js
const qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
```

或者用别的方法（ES 6）：

```js
import qs from 'qs';
const data = { 'bar': 123 };
const options = {
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  data: qs.stringify(data),
  url,
};
axios(options);
```

## Node.js

在 node.js 中，你可以使用 [`querystring`](https://nodejs.org/api/querystring.html) 模组：

```js
const querystring = require('querystring');
axios.post('http://something.com/', querystring.stringify({ foo: 'bar' }));
```

你也可以使用 [`qs`](https://github.com/ljharb/qs) 库。

> 注意
> 如果你需要序列化嵌套对象，使用 `qs` 库是更合适的，因为 `querystring` 方法有些已知的问题（[https://github.com/nodejs/node-v0.x-archive/issues/1665](https://github.com/nodejs/node-v0.x-archive/issues/1665)）

# 语义化版本

axios `1.0` 版本之后，新的 minor 版本会发布一些重大的变化。比如 `0.5.1` 和 `0.5.4` 有着同样的 API，但是 `0.6.0` 版本会有一些重要的改变。

# Promises

axios 依赖于原生 ES6 的 Promise。如果你的环境不支持 ES6 Promises，你可以 [polyfill](https://github.com/stefanpenner/es6-promise)。

# TypeScript

axios 包括一些 [TypeScript](http://typescriptlang.org/) 定义。

```typescript
import axios from 'axios';
axios.get('/user?ID=12345');
```