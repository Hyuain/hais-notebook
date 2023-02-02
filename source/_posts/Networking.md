---
title: 计算机网络
date: 2023-01-18 21:37:30
tags:
  - 入门
  - 计算机网络
categories:
  - [计算机]
---

计算机网络自顶向下方法 + 网课。

<!-- more -->

# 协议栈

- 应用层：FTP、DNS、HTTP
- 传输层：TCP、UDP
- 网络层：IP
- 链路层：DOCSIS
- 物理层：与双绞铜线、同轴电缆、光纤等相关

# 应用层

## HTTP

> Web 的应用层协议是超文本传输协议（HyperText Transfer Protocol, HTTP）

### HTTP 概述

#### HTTP 历史

- HTTP 0.9：1991，只有 GET、只能传HTML，没有 CSS、JS，每个HTTP请求都是短连接
- HTTP 1.0：1996，有了 POST、HEAD……
- HTTP 1.1：1997，目前为止最常用的版本
- HTTP 2.0：2015，HTTP 1.1 的扩展，于2015年5月提出
- HTTP 3.0：QUIC 协议（一种传输层协议，TCP的效率比较低，QUIC 为了减小 TCP 的延迟和带宽开销）

### HTTP 协议结构和通讯原理

#### HTTP 协议特点

- **支持客户/服务器模式**
- **简单快速**
    - 客户向服务器请求服务时，只需要传送请求方法和路径
    - GET、HEAD、POST
    - HTTP服务器程序规模小，因此通信速度块
- **灵活**
    - 允许传输任意类型的数据对象
	- 正在传输的类型由 Content-Type 加以标记
- **无连接**
    - 无连接的含义是限制 **每次连接只处理一个请求**
    - 服务器处理完客户的请求，并收到客户的应答后，即断开连接——keep alive 功能使得连接（TCP）不断开，有规定的超时时间
    - 采用这种方式可以节省传输时间
- **无状态**
    - 协议对事务的处理能力 **没有记忆能力**，每个请求都是 **独立的**
    - 如果后续处理需要前面的信息，则必须重传，导致每次连接的数据量增大
	- 另一方面，在服务器不需要先前信息时它的应答就较快

#### URI 和 URL

> 问题：我们在浏览器的 Web 地址应该叫 URL 还是 URI？
> - URI：可以分为 URL 和 URN，或同时具备 locators 和 names 特性的一个东西
> - URN 像一个人的名字，URL 像一个人的地址；URN 确定了东西的身份，URL 提供了找到它的方式（提供了访问机制，比如说协议）

URL 实际上是 URI 的一个子集，除识别资源外还提供定位资源的方法。 

下面是一个 URI 的组成：

```text
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
scheme  user information     host     port                  query         fragment

  urn:example:mammal:monotreme:echidna
  └┬┘ └──────────────┬───────────────┘
scheme              path
```

举一个我们常见的 URL（URI） 的例子：

https://zh.wikipedia.org/w/index.php?title=Special:随机页面#5

1. `https`：协议
2. `zh.wikipedia.org`：域名
3. `/w/index.php`：路径（不同的页面）
4. `?title=Special:随机页面`：查询参数（相同页面，不同内容）
5. `#5`：锚点（相同页面，相同内容，不同位置）
6. 其中若不写端口号，则表示使用 https 对应的默认端口号 443

#### HTTP 报文结构

##### 请求报文

```text
【请求行】POST（请求方法） /webTours/login.pl（请求 URI） HTTP/1.1（HTTP 协议及版本）
【请求头】用键值对来传递参数，包括 Host、Accept、Content-Type 等

【请求体】
```

可以用 curl 构造请求：

```bash
curl -X -POST # 设置请求动词
curl -H 'Accept: text/html' # -H 也可以写成 --header
curl -H 'Content-Type: text/plain;charset=utf-8' -d '请求体内容' # -d 也可以写成 --data
```

可以用 Node.js 读取请求：

```js
request.method
request.url // 路径，带参数
request.path // 纯路径
request.query // 只有查询参数
request.headers['Accept']
```

##### 响应报文

```text
【响应行】HTTP/1.1（报文协议及版本） 200 Ok（状态码及状态描述）
【响应头】

【响应体】
```

可以用 Node.js 设置响应：

```js
response.statusCode = 200
response.setHeader('Content-Type', 'text/html')
response.write('内容') // 内容可以追加
```

{% note warning %}
注意 `path` 都是以 `/` 开头的
{% endnote %}

#### HTTP 报文头

- **Accept**：浏览器端可以接收的媒体类型
    - **Accept: text/html**，代表浏览器可以接收服务器回发的类型为 text/html 类型的数据，如果服务器无法返回此类型，应该返回一个 406 错误（Non Acceptable）
    - **Accept: */***，代表浏览器可以处理所有类型
    - 可以设置优先级（权重值q，取0~1.000）
- **Accept-Encoding**：浏览器申明自己接收的编码（压缩）方法（gzip、deflate）
- **Accept-Language**：浏览器申明自己接收的语言
    - **Accept-Language**: zh-cn,zh;q=0.7,en-us,en;q=0.3 
- **Connection**
    - **Connection: keep-alive**，TCP 连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接
    - **Connection: close**，一个 Request 完成后，TCP 连接关闭
- **Host**：指定被请求资源的 Internet 主机和端口号，通常从 URL 中提取出来
- **Referer**：一般会带上此字段，告诉服务器是从哪个页面链接过来的
- **User-Agent**：告诉 HTTP 服务器，客户端使用的操作系统和浏览器名称和版本
- **Content-Type**：说明了报文体对象的媒体类型
    **text/html**：HTML
    **text/plain**：纯文本
    **text/xml**：XML
    **image/gif**：GIF图像
    **image/jpeg**：JPG图像
    **image/png**：PNG图像
    **application/xhtml+xml**：XHTML
    **application/xml**：XML
    **application/atom+xml**：Atom XML
    **application/json**：JSON
    **application/pdf**：PDF
    **application/msword**：WORD 
    **application/octet-stream**：二进制数据流
    **application/x-www-form-urlencoded**：表单提交

#### HTTP 请求方法

- **GET**：请求访问已被 URI 识别的资源
    - 提交的内容是 URL 的一部分，长度限制、安全性
- **POST**：与 GET 功能类似，一般用来传输实体的主体，主要目的是提交数据，不是获取响应主体的内容
- **PUT**：与 POST 最大的不同是，PUT 是幂等（不管重复多少次操作，都是实现相同的结果）的，POST 是不幂等的，因此一般创建对象用 POST，更新对象用 PUT，但是 PUT 没有验证机制，有安全性问题，所以一般还是用 POST
- **HEAD**：类似于GET，只不过返回的响应中没有具体的内容，用于获取报头（测试超链接的有效性）
- **DELETE**：请求删除资源，与 PUT 相反，并且没有验证机制，因此现在一般不用
- **OPTIONS**：用来查询针对请求 URI 指定的资源支持的方法
    - 服务器返回一个`Allow: GET, HEAD, POST`等等
- **TRACE**：回显服务器收到的请求，主要用于测试或诊断，但容易受到 XST 攻击，一般不用
- **CONNECT**：开启客户端与所请求资源之间的双向沟通的通道，可以用来创建隧道，一般用于 HTTP 代理

#### HTTP 状态码

![HTTP状态码](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/HTTP-StatusCode.png)

- **200 Ok**，请求已成功，并且所希望的响应头或数据体已经随此响应返回
- **202 Accepted**，已接收，但处理未完成
- **206 Partial Content**，部分内容，处理器成功处理了部分 GET 请求——断点续传
- **301 Moved Permanently**，永久移动，请求的资源已被永久移动到新的 URI，以后任何新的请求都应使用新的 URI 代替
- **302 Found**，临时移动，客户端应继续使用原有 URI
- **400 Bad Request**，客户端请求的语法错误，服务器无法理解
- **401 Unauthorized**，请求要求用户的身份验证
- **403 Forbidden**，服务器理解客户端的请求，但是拒绝执行此请求
- **404 Not Found**，服务器无法根据请求找到资源（网页）
- **500 Internal Server Error**，服务器内部错误，无法完成请求
- **502 Bad Gateway**，充当网关或代理的服务器，从远端服务器接收到了一个无效的请求

#### HTTP 状态管理：Cookie 和 Session

##### Cookie

W3C 推行的一种机制，客户端请求服务器，如果服务器需要记录该用户的状态，就向客户端浏览器颁发一个Cookie，
客户端浏览器会把Cookie保存起来，浏览器再请求的时候，就会把请求的网址连同Cookie一同提交给服务器

##### Session

另一种记录客户状态的机制，服务器把客户端信息以某种形式记录在服务器上
客户端可以以 Cookie、URL 重写或隐藏表单的形式保存 Session ID

![Session与Cookie](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Session-Cookie.png)

##### Cookie 和 Session

- 有效期不同
    - Cookie 的有效时间久，或者可以设置到永远
    - Session 的有效期
        - 超时自动失效，通常不长
		- 程序调用 `HttpSession.invalidate()` 主动失效（退出、注销等操作）
        - 服务器进程被终止
    - 存放位置不同，Cookie 在客户端，Session 在服务器端
	- 安全性（隐私策略）不同，用户可以更改 Cookie
	- 对服务器压力不同

### HTTP 协议的特性

#### HTTP 协议中的编码和解码

> 码 = 字符集+编码

##### 编码规范

- 字库表：里面存储了所有的字符
- 字符集：字符对应的二进制地址的集合
- 编码方式：一套编码规范可以有多种不同的编码方式（比如 UTF-8，对应的编码规范是 Unicode），一种算法来节约空间

常见的编码规范：

- ASCII 码：7 位码，128 个字符，1 个字节
- GBK：汉字内码扩展规范，2 个字节
- ISO-8859-1：加了希腊语等，把其他所有的当做 ISO-8859-1 来解都没问题，没有中文，8 位码，1 个字节
- Unicode：包含全世界所有的字符，最多 4 个字节

##### 乱码

乱码的由来：解码过程、编码过程都可能导致乱码

##### URL 的编码与解码

URL 是采用 ASCII 字符集进行编码的

`%` 编码规范：

- 对 URL 中属于 ASCII 字符集的非保留字不做编码
- 对 URL 中的保留字需要取其 ASCII 内码，然后加上%的前缀对该字符进行编码
- 对 URL 中非 ASCII 字符需要取其 Unicode 内码，然后加上 `%` 的前缀对该字符进行编码

> Fiddler

#### HTTP 协议的基本认证

##### 常见的认证方式

##### BASIC

![BASIC](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/BASIC.png)

不便捷灵活，且不安全（Base64实际上就是明文传输）

##### DIGEST

同样采用质询 / 响应方式，但不会明文传输密码

![DIGEST](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/DIGEST.png)

默认使用md5加密：
客户端发送响应摘要 = MD5(HA1:nonce:HA2)
其中，HA1 = MD5(username:realm:password), HA2 = MD5(method:digestURI)
虽然以前认为是不可逆的加密，但是仍然存在字典攻击、用户被冒充等风险

##### SSL 客户端

凭借客户端证书认证

##### FormBase

不是在 HTTP 协议中定义的，是使用 Web 应用各自实现的基于表单的认证，通过 Cookie 和 Session 来保持用户登录状态

#### HTTP 中的长连接和短链接

长连接又称持续连接（persistent connection），短连接又称非持续连接（non-persistent connection）

HTTP 中的长连接和短链接本质上是 TCP 的长连接和短链接
- HTTP/1.0 中，默认是短链接，每遇到一个外部资源就建立一个对话（创建一个 TCP 连接）
- HTTP/1.1 起，默认是长连接（多个请求共用一个 TCP 连接）

#### 中介代理与中介网关

##### 中介代理

![中介代理](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/HTTP-Proxy.png)

代理的作用：抓包、匿名访问、过滤器

##### 中介网关

![中介网关](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/HTTP-Getway.png)

扮演协议转换器的角色：
- (HTTP/) 服务器端网关，通过 HTTP 协议与客户端对话，通过其他协议与服务器通信
- (/HTTP) 客户端网关，通过其他协议与客户端对话，通过 HTTP 协议与服务器通信

常见网关类型：
- HTTP/*：服务器端 Web 网关
- HTTP/HTTPS：服务器端安全网关
- HTTPS/HTTP：客户端安全加速网关（SSL 卸载）
- 资源网关

#### HTTP 缓存

缓存的内容：CSS、JS、图片等更新频率不大的静态资源文件

##### HTTP 缓存头部字段

###### Cache-Control

> 请求/响应头，缓存控制字段

- no-store：所有内容都不缓存
- no-cache：缓存，但浏览器会请求服务器判断资源是否更新
- max-age=x：请求缓存后 x 秒不再发起请求
- s-maxage=x：类似，但只对 CDN 缓存有效
- public：客户端和代理服务器（CDN）都可以缓存
- private：只有客户端可以缓存

###### 其他头部字段

响应头，服务器返回：

- **Expires**：代表资源过期时间，是 HTTP/1.0 的属性，比 HTTP/1.1 的 **Cache-Control:max-age=x** 优先级低
- **Last-Modified**：资源最新修改时间
- **Etag**：缓存资源标识

请求头，服务器提供：

- **if-Modified-Since**：资源最新修改时间，与 **Last-Modified** 是一对，他们会进行对比
- **if-None-Match**：缓存资源标识，与 **Etag** 是一对（其实就是上次服务器给的 Etag），他们会进行对比

##### HTTP 缓存工作方式

###### 场景一：让服务器与浏览器约定一个文件过期时间 Expires

服务器给浏览器一个 Expires，后续请求浏览器会对比当前 **本地时间** 是否已经大于 Expires，超过过期时间再请求。

问题：即使超过过期时间之后，文件可能仍然没有变化

###### 场景二：约定 Expires 的基础上，再通过文件的最新修改时间进行对比 Last-Modified 与 if-Modified-Since

服务器给浏览器 Expires 和 Last-Modified，后续若超过 Expires 后，浏览器请求时会带上 if-Modified-Since，服务器进行对比，如果文件未修改，就返回 304 Not Modified，让浏览器使用缓存

问题：浏览器可以随意修改本地时间，而且 Last-Modified 只能精确到秒

###### 场景三：在上面的基础上加上 Etag/If-None-Match，再使用 max-age

max-age 使用的是 **相对时间**，因此浏览器不能通过修改本地时间的方式来影响缓存，优先级高于 Expires；
Etag 优先级也高于 Last-Modified，因为文件只要修改过，Etag 就会发生变化

至此其实 Last-Modified 和 Expires 其实已经没什么用了，但很多时候还是会加上

问题：max-age 或者 Expires 未过期的情况下，若改动文件，应该怎样让浏览器知道？

##### 缓存改进方案

###### md5/hash 缓存

不缓存 html，为静态文件添加 MD5 或者 hash 标识，比如将静态资源的文件名改为 `f-hash1.js`，若文件修改了，文件名也会改变为 `f-hash2.js`

###### CDN 缓存

> CDN 是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率

CDN 的作用：
1. 减少源站压力
2. 解决跨区域访问问题

CDN缓存的工作方式：
1. 第一次请求，CDN 和 浏览器同时缓存
2. 浏览器缓存过期后，找 CDN 进行对比，CDN 看自己的过期没有，若自己没过期，就给发给浏览器

##### 浏览器操作对 HTTP 缓存的影响

| 用户操作 | Expires/Cache-Control | Last-Modified/Etag |
| --- | --- | --- |
| 地址栏回车 | 有效 | 有效 |
| 页面链接跳转 | 有效 | 有效 |
| 新开窗口 | 有效 | 有效 |
| 前进、后退 | 有效 | 有效 |
| F5 刷新 | *无效* | 有效 |
| Ctrl+F5 刷新 | *无效* | *无效* |

### HTTP 内容协商机制

> 指客户端和服务端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源。内容协商会以相应资源的语言、字符集、编码方式等作为判断的基准

#### 内容协商的方式

- **客户端驱动**：客户端发起请求，服务器发送可选项列表，客户端做出选择后再发送第二次请求
- **服务器驱动**：服务器检查客户端的请求头部集并决定提供哪个版本的页面
    - 客户端发送：Accept、Accept-Language、Accept-Charset、Accept-Encoding
        - `Accept-Language: en;q=0.5, fr;q=0.0, nl;q=1.0, tr;q=0.0`
    - 服务器返回：Content-Type、Content-Language、Content-Type、Content-Encoding
- **透明协商**：某个中间设备（通常是缓存代理）代表客户端进行协商

### HTTP 断点续传和多线程下载

主要是通过请求头中的 **Range** 和响应头中的 **Content-Range** 实现的，并且若使用断点续传模式返回的状态码都是 206 Partial Content

#### Range

用于请求头中，指定第一个字节的位置和最后一个字节的位置：

```text
Range:(unit=first byte pos)-[last byte pos]
Range:bytes=0-499
Range:bytes=-500
Range:bytes=500-
Range:bytes=500-600,601-999
```

#### Content-Range

用于响应头中，返回当前接受的范围和文件总大小：

```text
Content-Range:bytes(unit first byte pos)-[last byte pos]/[entity length]
```

### HTTPS

#### 数字证书

##### 对称加密与非对称加密

![对称加密](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Encryption.png)

![非对称加密](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/AsymEncryption.png)

对称加密效率高，但在密钥分发时存在安全隐患，容易被截获；非对称加密安全性高，但性能低下；因此我们可以先用非对称加密传输对称加密的密钥，之后再用对称加密来传输数据

##### 数字签名与数字证书

![数字签名](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Signature-1.png)

![数字签名](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Signature-2.png)

![数字证书](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Certificate-1.png)

![数字证书](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Certificate-2.png)

#### HTTPS 协议概述

> HTTPS 可以认为是 HTTP + TLS，TLS 是传输层加密协议，他的前身是 SSL 协议

可以认为 TLS 建立在传输层和应用层之间（会话层），目前常用的版本有 TLS/1.0、1.1、1.2、1.3 和 SSL/3.0，但是 SSL/3.0 可能会存在 ‎POODLE 攻击，TSL/1.0 也存在一些漏洞

HTTPS 功能：
    - **内容加密**（非对称加密协商密钥 + 对称加密传输数据）
    - **数据完整性**（数字签名）
    - **身份认证**（数字证书）

#### HTTPS 使用成本

- 证书费用以及更新维护
- 降低用户的访问速度
- 消耗 CPU 资源

HTTPS 对性能的影响：

- 协议交互所增加的网络往返时延（Round-Trip Time）
    - 有可能用户访问 HTTP，服务器需要返回 302 使其跳转到 HTTPS
    - TLS 完全握手阶段 1 和 2，以及与 CA 服务器进行连接与验证
- 加解密相关的计算耗时
    - 浏览器计算耗时
    - 服务端计算耗时
    

{% note warning %}
HTTPS 并不能解决所有的劫持问题
{% endnote %}

### 基于 HTTP 的功能追加协议

#### HTTP 协议的瓶颈

- 单路连接、请求低效，一条连接上只可发送 **一个** 请求，并且严格先入先出（非关键资源阻塞问题）
- 请求只能 **从客户端开始**，客户端不可以接受除了响应以外的指令，没办法让服务器一更新，客户端就立即更新
- 头部冗余，请求/响应头部 **不经压缩** 就发送
- 每次相互发送 **相同的头部** 造成浪费
- 不强制使用加密

#### WebSocket

> 是为了解决 HTTP 长连接问题而做出的改良协议，与 HTTP 协议有交集

HTTP 的生命周期是由 Request/Response 确定的，一个 Request 与 一个 Response 对应，且 Response 是被动的。

WebSocket 是持久化协议，

一个典型的 WebSocket 握手：

```text
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==   浏览器随机生成的 Base64
Sec-WebSocket-Protocol: chat, superchat  用来区分同一个 URL 下不同的服务所需要的协议 
Sec-WebSocket-Version: 13
Origin: http://example.com
```

传统的轮询方式：

- AJAX 轮询：浏览器每隔几秒发一次请求，询问服务器有没有新消息
- 长轮询（Long Poll）：采取阻塞的方式，客户端发起连接，如果没有消息，服务器就一直不返回，直到有消息再返回

> 传统方式问题：太占用资源了，AJAX 轮询需要很高的处理速度，长轮询则需要能容纳很高的并发数

WebSocket：

- 一次连接，长期有效，服务器有消息的时候再推送（回调）

![WebSocket](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/WebSocket.png)

WebSocket 的主要特点：

- 真正的 **全双工方式**，允许服务器向客户端主动推送数据
- 减少 **通信量**

#### SPDY

> 是谷歌开发的基于 TCP 的应用层协议，为了降低网络延迟，优化用户体验，也是对 HTTP 协议的增强

![SPDY](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SPDY.png)

SPDY 的改进：

- 多路复用、请求优化，允许多个请求共用一个 TCP 连接，并且可以设置优先级（避免非关键资源阻塞）
- 支持服务器推送技术，主要是资源类的推送（比如若浏览器请求了 `style.css`，服务器就主动再预推送一个 `style.js`），跟 WebSocket 不同
- 压缩了 HTTP 头
- 强制使用 SSL 传输协议

#### HTTP/2.0

> 可以理解为 SPDY 的升级版，基本目标之一是减少传送单一 Web 页面时的并行 TCP 连接，减少需要服务器需要打开与维护的套接字数量。并且要求仔细设计相关机制避免队首（Head Of Line, HOL）阻塞。

##### 二进制分帧

> HTTP/2.0 性能增强的核心，解决 HOL 阻塞。

![二进制分帧](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Binary-Framing.png)

在二进制分帧层上，会将原来传输的信息分成更小的帧，并且采用二进制编码；HTTP/2.0 的通信都在一个连接上完成，这个连接可以承载任意数量的双向数据流，每个数据流都以消息的形式发送，这些消息由一个或多个帧组成，帧可以乱序发送，最后会根据每个帧上面的流标识符重新组装。

##### 首部压缩

![首部压缩](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Header-Compression.png)

使用 **首部表** 来跟踪和存储键值对，相同的数据不再通过每次响应和请求发送，通讯期间几乎不会改变 **通用的键值对**，每次只会发送新增或改变的部分头部，首部表在整个连接中有效，会不断更新

##### 多路复用

![多路复用](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Multiplex.png)

所有通信都在一个 TCP 连接上完成，将 HTTP 协议通信的基本单位缩小为帧，让多个数据流共用一个连接，提高利用率

单链接多资源的优势：

- 可以 **减少服务器连接压力**，内存占用少了，连接吞吐量大了
- 由于 TCP 连接减少而使 **网络拥塞状况** 得以改观
- 慢启动时间减少，**拥塞** 和 **丢包** 恢复速度更快

##### 并行双向字节流

![并行双向字节流](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/HTTP2-1.png)

- 并行交错地发送请求，请求之间互不影响
- 并行交错地发送响应，响应之间互不影响
- 只使用一个连接即可并行发送多个请求和响应
- 消除不必要的延迟，减少页面加载的时间

##### 请求优先级

- 可以为每个请求报文分配 1~256 的权重，数字较大说明拥有更高的优先级
- 可以通过指明相关报文段的 ID，说明报文段之间的关联性

##### 服务器推送

![服务器推送](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/HTTP2-2.png)

允许服务器为一个请求发送多个响应

#### WebDAV

> 基于万维网的分布式创作和版本控制，用于管理 Web 服务器的文件

![WebDAV](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/WebDAV.png)

#### QUIC 和 HTTP/3.0

![QUIC和HTTP/3.0](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-HTTP3.0.png)

HTTP/2.0 的问题：

- 队首阻塞，如果出现丢包，整个 TCP 连接都需要停下来重传，因为 HTTP/1.1 是建立多个 TCP 连接，所以不会全部都需要停下来
- 建立连接的握手延迟大，TCP 需要握手，对于短连接场景影响大

QUIC 的特性：

- **0 RTT**
![QUIC建立连接的过程](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-1.png)
- **没有队头阻塞的多路复用**
![TCP队头阻塞](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Blocking.png)
![QUIC没有队头阻塞](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-No-Blocking.png)
- **前向纠错**，会多发一些冗余的包（校验包），虽然产生了多的包，但是减少了重传带来的损失
![前向纠错](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-Error-Correcting.png)



## 电子邮件

- 因特网电子邮件系统总体由三个部分组成：**用户代理（User Agent）、邮件服务器（Mail Server）、简单邮件传输协议（Simple Mail Transfer Protocol, SMTP）**。

- 每个接收方在邮件服务器上有一个 **邮箱（Mailbox）**。
- 发送方的用户代理传输到发送方的邮件服务器，然后被分发到接收方的邮箱中。

```text
A的代理 --SMTP/HTTP--> A的邮件服务器 --SMTP--> B的邮件服务器 --HTTP/IMAP--> B的代理
```



### 推送报文（SMTP）

- 发送方的代理（手机、电脑等）可以通过 SMTP 向邮件服务器推送报文。
- 发送方的邮件服务器通过 SMTP 向接收方邮件服务器推送报文。
- 一般不使用中间邮件服务器，而是由 **发送方邮件服务器**（STMP Client） 与 **接收方邮件服务器**（STMP Server） 直接建立的 **TCP** 连接。
- STMP Client 与 Server 在 25 端口建立 TCP 连接之后，可以发送命令：HELO、MAIL FROM、RCPT TO、DATA、QUIT。

#### 邮件报文格式（DATA）

```text
From: xxx@server.com
To: yyy@another.com
Subject: xxxxxx
```

### 获取报文

- IMAP
- HTTP
- POP3

## DNS

> DNS (Domain Name System) 是：
>
> 1. 一个由分层的 DNS 服务器（通常是运行 BIND (Berkeley Internet Name Domain) 的 UNIX 机器）实现的分布式数据库
> 2. 一个使得主机能够查询分布式数据库的应用层协议（使用 53 端口的 UDP）

DNS 的作用：

- 进行主机名到 IP 的转换
- 主机别名（host aliasing）
- 邮件服务器别名（mail server aliasing）
- 负载分配（load distribution）

### DNS 的分布式设计

#### 分布式、层次数据库

- **根 DNS 服务器**。全球有超过 1000 台根服务器实体，他们是 13 个不同的根服务器副本，由 12 个不同的组织管理。根服务器提供 TLD 服务器的 IP 地址。
- **顶级域（Top-Level Domain, TLD）服务器**。每个顶级域（com、org、edu 等）都有 TLD 服务器或服务器集群。TLD 服务器提供了权威 DNS 服务器的 IP 地址。
- **权威 DNS 服务器**。多数大学和公司实现并维护他们自己的基本和辅助（备份）的权威 DNS 服务器。

- **本地 DNS 服务器**。严格来说并不属于 DNS 服务器层次结构，一般一个居民区的 ISP 或一个机构的 ISP 都有一台本地 DNS 服务器。

##### DNS 服务器的交互

下图利用了 **递归查询（recursive query）**和 **迭代查询（iterative query）**。从 cis.poly.edu 到 dns.poly.edu 是递归查询，因为该查询以自己的名义请求，后续的 3 个查询是迭代查询。

![Networking-DNS-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-DNS-1.jpg)

而下图中的所有查询都是递归查询，我们通常遵循上图的查询模式：

![Networking-DNS-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-DNS-2.jpg)

#### DNS 缓存

事实上，因为缓存，只有很少部分 DNS 查询需要用到根服务器。

### DNS 记录

- DNS 服务器存储了 **资源记录（Resource Record, RR）**
- 资源记录是包括了下列字段的 4 元组：`(Name, Value, Type, TTL)`
  - TTL 是缓存时间
  - 如果 Type = A，则 Name 是主机名， Value 是 IP 地址，比如 `(relay1.bar.foo.com, 145.37.93.126, A)`
  - 如果 Type = NS，则 Name 是域，Value 是权威 DNS 服务器的主机名，比如 `(foo.com, dns.foo.com, NS)`
  - 如果 Type = CNAME，则 Name 是主机别名，Value 是规范主机名，比如 `(foo.com, relay1.bar.foo.com, CNAME)`
  - 如果 Type = MX，则 Name 是主机别名，Value 是邮件服务器的规范主机名，比如 `(foo.com, mail.bar.foo.com, MX)`。通过使用 MX 记录，公司的邮件服务器可以和其他服务器使用相同的别名

### DNS 报文

DNS 只有查询和回答报文，并且两种报文有相同的格式：

![Networking-DNS-3](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-DNS-3.jpg)

可以使用 `nslookup` 直接向 DNS 服务器发送查询报文。

## 视频流和内容分发网

### HTTP 流和 DASH

- **HTTP 流**：视频只是一个普通的文件，每个文件有一个特定的 URL。接收到视频就进行播放，同时缓存该视频后面部分的帧。
  - 缺陷：所有客户端接收到相同编码的视频
- **DASH (Dynamic Adaptive Stream over HTTP)**：视频编码为几个不同的版本，客户动态地请求来自不同版本且长度为几秒的视频段数据块。每个视频版本存储在服务器中，HTTP 服务器有一个告示文件（manifest file），为每个版本提供一个 URL 和比特率。

### 内容分发网

> 内容分发网即 CDN (Content Distribution Network)

- CDN 管理分布在多个地理位置的服务器，在他的服务器中存储各种 web 内容的副本，并且将每个用户请求定向到一个将提供最好用户体验的 CDN 位置
- CDN 可以是专用 CDN（private CDN），也可以是第三方 CDN（third-party CDN）

# 运输层

## 多路复用与多路分解

> 运输层是怎样将网络层的报文段发送给应用层的不同进程的

- 一个进程拥有一个或多个 **套接字（socket）**
- 运输层并没有直接将数据交付给进程，而是交给了某一个套接字
- **多路分解（demultiplexing）**：将运输层报文段中的数据交付给正确的套接字
- **多路复用（multiplexing）**：从不同套接字中收集数据块，并封装上首部信息从而生成报文段，然后发送给网络层

- 用于区分套接字的特殊字段就是端口号，下面是一个典型的运输层报文段的结构：

![Networking-Transport-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-Transport-1.jpg)

### 无连接和面向连接

- UDP 的套接字是由**（目的 IP 地址，目的端口号）**全面标识的，不同源但相同目的的报文段会被扔给同一个套接字。
- TCP 的套接字是由 **（源 IP 地址，源端口号，目的 IP 地址，目的端口号）**标识的，因此不同源的报文段会被扔给不同的套接字。

## UDP

### 报文结构

![Networking-UDP-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-UDP-1.jpg)

### UDP 校验和（checksum）

- 校验和：发送方将报文段中的 16 bit 字的和进行反码运算
  - 如果有溢出，需要进行反卷（即将溢出的高位与剩下的部分做加法运算）
- 接收方将全部的 16 bit 字（包括校验和）加在一起，得到的结果应该是 `1111 1111 1111 1111`，如果其中有 0，则代表出现了问题

- UDP 提供运输层的差错检测，在系统设计中被称为 **端到端原则（end-end principle）**。因为假定 IP 是可以运河在任何第二层协议之上的，UDP 相当于提供了一层保险，但注意 UDP 无法恢复差错。

## 可靠数据传输

> 在 **不可靠的下层协议上** 通过 **可靠数据传输协议（reliable data transfer protocol）** 为 **上层实体** 提供 **可靠数据传输信道**。

### 什么都不考虑：rdt1.0

![Networking-rdt-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-1.jpg)

### 考虑比特差错：rdt2.0

> 校验和、序号、ACK、重传

通常使用 **自动重传请求（Automatic Repeat reQuest, ARQ）**协议来解决这个问题，ARQ协议中还需要三种功能来处理比特差错：
- 差错检测
- 接收方反馈（ACK、NAK）
- 重传

![Networking-rdt-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-2.jpg)

**问题：无法解决 ACK 和 NAK 分组受损的情况。**

可以引入序号 0 和 1（因为还没有考虑到丢包的情况，这个 1 bit 的序号就够用了），来确认是正在重传之前的分组、还是发送了一个新的分组。

此外，引入序号之后，也可以用 ACK + 序号 来代替 NAK 了，如果发送方接收到 ACK + 之前的序号，说明当前的包没有被正确接收到，则需要重传当前的包。

![Networking-rdt-3](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-3.jpg)

![Networking-rdt-4](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-4.jpg)

### 考虑比特差错和丢包：rdt3.0

> 定时器

由于分组序号在 0 和 1 之间交替，因此又被称为 **比特交替协议（alternating-bit protocol）**

![Networking-rdt-5](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-5.jpg)

**问题：以上讨论的实际上都是停等（stop-and-wait）协议，发送方需要收到正确的 ACK 才能做出下一步行动，否则就会一直卡在等待状态。**

### 考虑流水线数据传输

与停等相对应的就是流水线，分组可以看做被填充到一条流水线中，源源不断地发送过去。因此需要考虑到如下的影响：

- **增加序号范围**。每个传送的分组必须有唯一序号，因为可能有多个在传输中的未确认报文
- **可能需要缓存多个分组**。
- **考虑解决流水线上丢失、损坏、延时过大的问题**。两种基本方法：**GBN (Go-Back-N)**，**选择重传（Selective Repeat, SR）**。

#### GBN (Go-Back-N)

> GBN 协议也常被称为 **滑动窗口协议（sliding-window protocol）**

- 基序号 base：最早未确认分组的序号
- 窗口大小 N：当接收到应用层序号大于 base + N 的数据包时，拒绝发送此数据包
- 累计确认：发送方收到序号为 n 的 ACK 时，认为 n 及 n 以前的都已经被

![Networking-rdt-6](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-6.jpg)

 ![Networking-rdt-7](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-7.jpg)

![Networking-rdt-8](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-8.jpg)



# TCP

## TCP 与 UDP

> TCP 面向连接、可靠、面向字节流；UDP 面向无连接

1. **面向连接**。TCP 需要三次握手建立连接，UDP 则不需要
2. **可靠**。TCP **有状态**，会记录哪些数据发送了，哪些数据对方接受了，哪些数据丢失了，不允许差错；TCP **可控制**，会根据丢包的情况或者状态，调整自己的行为，比如控制速度或者重发
3. **面向字节流**。UDP 的数据传输是面向数据报的，TCP 为了维护状态，将 IP 包变成了字节流。

所谓 **面向数据报** 是指，应用层交给 UDP 多长的 **报文**，UDP 就照样发送，一次发送一个报文；而面向字节流是指，TCP 虽然一次接收应用层来的一个数据块（大小不等），但 TCP 把应用程序看成一连串无结构的字节流。TCP 有一个缓冲，当应用程序传送的数据块太长，TCP 就会把它划分得短一些再发送；如果应用程序一次只发送一个字节，TCP 也可以等待积累足够多的字节后再构成报文段发送出去。

## TCP 的三次握手与四次挥手

### 三次握手

> 三次握手是需要确认双方的 **发送能力** 和 **接收能力**

![TCP三次握手](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Handshake.png)

#### 三次握手的过程中可以携带数据吗？

前两次不能，第三次可以，此时客户端已经处于 ESTABLISHED 状态，确认了服务器发送、接受信息的能力正常

### 四次挥手

![TCP四次挥手](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Wavehand.png)

服务端在接收到 FIN, 往往不会立即返回 FIN, 必须等到服务端所有的报文都发送完毕了，才能发 FIN。因此先发一个 ACK 表示已经收到客户端的 FIN，延迟一段时间才发 FIN。这就造成了四次挥手。

### 半连接队列与 SYN Flood

三次握手之前，在服务器状态从 CLOSED 变为 LISTEN 的时候，还在内部创建了两个队列：**半连接队列** 和 **全连接队列**

#### 半连接队列

又称 **SYN 队列**，当客户端发送 SYN 到服务端，服务端收到后回复 ACK 和 SYN，状态由 LISTEN 变为 SYN_RCVD，此时这个连接就被推入了 **半连接队列**

#### 全连接队列

又称 **ACCEPTED 队列**，当客户端返回 ACK，服务端接收后，三次握手完成，连接等待被某个具体的应用取走，在被取走之前，会被推入 **全连接队列**

#### SYN Flood 攻击

属于典型的 Dos/DDos 攻击，用客户端在短时间内伪造大量不存在的 IP 地址，疯狂发送 SYN，使得服务器：
1. 处理大量 SYN 包并返回 ACK，使得 **半连接队列爆满**
2. 由于是不存在的 IP，服务端长时间 **收不到客户端的 ACK**，就会疯狂重发数据，直到耗尽服务器资源

##### 如何应对 SYN Flood 攻击？

1. **增加半连接队列容量**
2. **减少 SYN + ACK 的重试次数**
3. **利用 SYN Cookie 技术**，在服务端接收到 SYN 后不立即分配链接资源，而是根据 SYN 计算出一个 Cookie，连通第二次握手一同发给客户端，客户端回复 ACK 的时候带上 Cookie，服务端验证合法后才分配链接资源

## TCP 报文

### TCP 报文头

![TCP报文头](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Header.png)

#### 序列号

即 Sequence Number，指的是本报文段的第一个字节的序列号，长度为 4 个字节，可以：

1. 在 SYN 报文中交换彼此的初始序列号
2. 保证数据包按照正确的顺序组装

##### 初始序列号

即 Initial Sequence Number，在三次握手的过程中，双方会通过 SYN 报文来交换彼此的 ISN，ISN 会动态增长，每 4ms 就加 1，溢出则回到 0。动态增长的 ISN 增加了猜测 ISN 的难度

#### 确认号

即 ACK（Acknowledgement Number），用来告知对方下一个期望接收的序列号，小于 ACK 的所有字节已经全部接收到

#### 标记位

占用一个字节，常见的标记位有 SYN、ACK、FIN、RST、PSH。

- SYN：Synchronize，表示同步，用来开始请求
- ACK：Acknowledge，表示确认
- FIN：Finish，表示发送方准备断开连接
- RST：Reset，用来强制断开连接
- PSH：Push，告知对方这些数据包收到后应该马上交给上层的应用，不能缓存

#### 窗口大小

占用两个字节，但实际上不够用，因此 TCP 引入了窗口缩放的选项，作为窗口缩放的比例因子，这个比例因子的范围在 0 ~ 14，比例因此可以将窗口值扩大为原来的 2 ^ n 倍

#### 校验和

占用两个字节，防止传输过程中数据包有损坏，如果遇到校验和有差错的报文，TCP 直接丢弃、等待重传

#### 可选项

![可选项](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Header-Options.png)

常用的可选项有：
- Timestamp：TCP 时间戳
- MSS：TCP 允许的从对方接收的最大报文段
- SACK：选择确认选项
- Window Scale：窗口缩放

##### Timestamp

Timestamp 一共占 10 个字节：

```text
kind(1 Byte) + length(1 Byte) + info(8 Bytes)
```

其中 `kind=8` `length=10`，`info` 由 `timestamp` 和 `timestamp echo` 两部分构成，各占四个字节

TCP 的时间戳主要解决两个问题：
- 计算往返时延（RTT, Round-Trip Time）
- 防止序列号回绕

###### 计算 RTT

![计算RTT](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Timestamp-RTT.png)

###### 防止序列号回绕

因为序列号的长度是有限的，若超出容纳的范围，就会回到 0，时间戳使得两个序列号相同的报文也能得以区分

## TFO

> TCP Fast Open，优化后的 TCP 三次握手流程，也是基于 SYN Cookie 技术

### TFO 流程

首轮的三次握手：

1. 客户端发送 **SYN** 给服务端
2. 服务端计算得到一个 **SYN Cookie**，将这个 Cookie 放到 TCP 报文的 Fast Open 选项中，再继续三次握手
3. 客户端把 **Cookie** 缓存下来，继续三次握手

后续的三次握手：

1. 客户端将之前缓存的 **Cookie、SYN 和 HTTP 请求** 发送给服务端
2. 服务端验证 Cookie 的合法性，如果合法就正常返回 **SYN + ACK**，同时可以给客户端发 **HTTP 响应**

![TFO](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TFO.png)

## TCP 超时重传

> 在间隔一段时间没有等到这个数据包回复时，将会重传这个数据包

### 超时重传时间的计算

重传的间隔被称为超时重传时间（RTO, Retransmission TimeOut），他的计算跟 RTT 密切相关，有两种主要的方法：经典方法和标准方法

#### 经典方法

> 平滑往返时间（SRTT, Smoothed Round-Trip Time）是根据 RTT 和之前的 SRTT 计算出来的一个值：

```text
SRTT = (α * SRTT) + ((1 - α) * RTT)
初始的 SRTT 为 0
```

α 即为 **平滑因子**，建议值是 0.8，范围是 0.8 ~ 0.9

```text
RTO = min(ubound, max(lbound, β * SRTT))
```

其中，β 是 **加权因子**，一般为 1.3 ~ 2.0，**lbound 是 下界**，**ubound 是上界**

在 RTT 变化较大的地方不太适用，因为 RTT 对 RTO 影响太小

#### 标准方法

又称 Jacobson / Karels 算法

1. **计算 SRTT**：

```text
SRTT = (α * SRTT) + ((1 - α) * RTT)
```

α 的建议值是 **0.125**

2. **计算中间变量 RTTVAR（Round-Trip Time Variation）

```text
RTTVAR = (1 - β) * RTTVAR + β * (|RTT - SRTT|)
```

β 的建议值是 *0.25**

3. 计算 RTO

```text
RTO = µ * SRTT + ∂ * RTTVAR 
```

µ 建议值取 1, ∂ 建议值取 4

### TCP 流量控制

TCP 会把需要发送的数据放到 **发送缓存区**，需要接收的数据放到 **接收缓存区**

流量控制则是通过 **控制接收缓存区的大小**，来控制发送端的发送，如果接收方的接收缓存区满了，就不能再发送了

#### 滑动窗口

TCP 滑动窗口分为 **发送窗口** 和 **接收窗口**

#### 发送窗口

![TCP发送窗口](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Send-Window.png)

![TCP发送窗口](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Send-Window-2.png)

SND 即为 send，WND 即为 window，UNA 即为 unacknowledged，NXT 即为 next

#### 接收窗口

![TCP接收窗口](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Receive-Window.png)

REV 即为 receive

简单来说，接受端如果处理能力不够用了，想让发送端少发点，就会将自己的接收窗口大小（相当于缓冲队列，也就是还可以装的部分）在 ACK 报文首部加上，发给发送端，发送端收到之后调解自己发送窗口的大小

### TCP 拥塞控制

流量控制主要是考虑到 **发送端** 和 **接受端** 的情况，而没有考虑到整个网络环境的影响，拥塞控制则是考虑整体网络环境的好坏

对于拥塞控制，TCP 每条连接都需要维护两个核心状态：
- 拥塞窗口（CWND, Congestion Window）
- 慢启动阈值（SSThresh, Slow Start Threshold）

涉及这几个算法：
- 慢启动
- 拥塞避免
- 快速重传和快速恢复

#### 拥塞窗口

指的是目前自己还能传输的数据量大小，与上面的接收窗口不同，是 **发送端** 的限制

发送窗口最终的大小取 **接收窗口** 和 **拥塞窗口** 中的小值

```text
发送窗口大小 = min(RWND, CWND)
```

### 慢启动

由于刚接入网络传输数据的时候，并不知道网络的状态，所以不能操之过急，拥塞控制会采用保守的算法来慢慢适应网络

- 首先，三次握手，双方宣告自己 **接收窗口（RWND）** 的大小
- 双方初始化自己 **拥塞窗口（CWND）** 的大小
- 在开始传输的一段时间，发送端每收到 1 个 ACK，拥塞窗口大小 **+ 1**，也就是说，每经过 1 个 RTT，CWND 翻倍，直到 CWND 达到 **慢启动阈值**

### 拥塞避免

当 CWND 达到慢启动阈值时，CWND 每次只能 **+ 1 / CWND**，也就是说，1 个 RTT 之后，CWND 只能增加 1

### 快速重传与快速恢复

#### 快速重传

解决的是 **是否需要重传** 的问题

当接受端发现丢包后，后续 ACK 一律回复为最后一个好包，因此当发送端发现接收到重复的 ACK，意识到丢包，就马上重传，而不是等一个 RTO

比如第 5 个包丢了，即使第 6、7 个包到达的接收端，接收端也一律返回第 4 个包的 ACK。当发送端收到 3 个重复的 ACK 时，意识到丢包了，于是马上进行重传

#### 选择性重传

解决的是 **如何重传的问题**

接受端会回复一个 ACK 报文，在首部可选项中有 **SACK** 属性，通过 **left edge** 和 **right edge** 告知发送端已经接收到了哪些区间的数据包，进行 **选择性重传（SACK, Selective Acknowledgement）**，比如上面的例子，不会再传第 6、7 个包

#### 快速回复

当发送端知道丢包之后，觉得网络可能有点拥塞，即会进入 **快速恢复** 阶段，发送端会做出如下改变：

- 拥塞阈值降为 CWND 的一半
- CWND 的大小变为拥塞阈值
- CWND 线性增加

## Nagle 算法与延迟确认

### Nagle 算法

> 为了避免频繁发送小包而诞生

- 第一次发送数据的时候不用等待，即使是再小的包也马上发送
- 后面发送的包需要满足下面条件之一：
    - 数据包大小达到最大子段大小（MSS, Max Segment Size）
    - 之前所有包的 ACK 都已经接受到

### 延迟确认

> 为了避免频繁回复小包而诞生

TCP 要求延迟的时延必须小于 500ms，一般的操作系统都不会超过 200ms

一些场景收到之后会马上回复：
- 收到了大于一个 frame 的报文，且需要调整窗口大小
- TCP 处于 quickack 模式（通过 `tcp_in_quickack_mode` 设置）
- 发现了乱序包

## TCP 的 keep-alive

由于 TCP 不是一个轮询的协议，他无法获知对端连接失效的情况（比如对端网络故障或宕机），**keep-alive** 则是用于探测对端的连接是否失效，但是因为探测的时间比较长，所以一般不怎么用

##### DNS 解析

- 系统在 HOST 文件中找 IP 地址
- 系统向本地 DNS 服务器请求：114.114.114.114（国内移动、电信和联通通用的DNS），8.8.8.8（GOOGLE 公司提供的 DNS，该地址是全球通用）
- 本地 DNS 服务器一层层向上请求，直到根 DNS 服务器
