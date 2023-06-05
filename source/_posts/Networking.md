---
title: Networking
date: 2023-01-18 21:37:30
categories:
  - [计算机]
mathjax: true
---

本文几种了计算机网络的相关知识，主要包括协议栈介绍、应用层、传输层和网络安全相关的知识，本文将会不断扩充积累。

目前，按照目录框架看大概分为以下几个部分：

- 概述，包括协议栈及其他基本概念
- 应用层，包括 HTTP、HTTPS、Email、DNS、视频流、CDN 等
- 传输层，包括多路复用与多路分解、UDP、可靠数据传输、TCP、TCP 连接管理及其他性能优化
- 网络安全

<!-- more -->

# Introduction

## Switching

- **电路交换（Circuit Switching）**：主要用于电话系统，用户之间会建立一条专用的物理链路，并且在通信过程中始终占用该链路。电路交换的线路利用率通常很低。
- **分组交换（Packet Switching）**：则是一个相对的概念，分组包含首部和尾部，其中包含了源地址和目的地址等控制信息，同一条线路上的多个分组不会相互影响。分组交换又像邮局一样，收到一些分组后会存储下来，然后把一些相同目的地的分组一起发送给下一个目的地。

## Delay

> 总时延 = 排队时延（Queuing Delay）+ 节点处理时延（Nodal Processing Delay）+ 传输时延（Transmission Delay）+ 传播时延（Propagation Delay）

- **排队时延（Queuing Delay）**：分组在路由器的输入和输出队列中等待的时间，取决于当前网络的**通信量（Traffic）**。

- **节点处理时延（Nodal Processing Delay）**：主机或路由器收到分组时进行处理所需要的时间，例如分析首部、从分组中提取数据、检验差错、查找路由等。

- **传输时延（Transmission Delay）**：主机或路由器传输数据帧所需要的时间。其中 $l$ 是数据帧的长度，$v$ 是传输速率。
  $$
  d = \frac{l(bit)}{v(bit/s)}
  $$

- **传播时延（Propagation Delay）**：电磁波在信道中传播需要花费的时间。其中 $l$ 是信道长度，$v$ 是电磁波在信道中传播的速度。
  $$
  d = \frac{l(m)}{v(m/s)}
  $$

## Protocol Stack

- 应用层：FTP、DNS、HTTP
- 传输层：TCP、UDP
- 网络层：IP
- 链路层：DOCSIS
- 物理层：与双绞铜线、同轴电缆、光纤等相关

# Application Layer

## HTTP

> Web 的应用层协议是超文本传输协议（HyperText Transfer Protocol, HTTP）

### HTTP History

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
    - **text/html**：HTML
    - **text/plain**：纯文本
    - **text/xml**：XML
    - **image/gif**：GIF图像
    - **image/jpeg**：JPG图像
    - **image/png**：PNG图像
    - **application/xhtml+xml**：XHTML
    - **application/xml**：XML
    - **application/atom+xml**：Atom XML
    - **application/json**：JSON
    - **application/pdf**：PDF
    - **application/msword**：WORD 
    - **application/octet-stream**：二进制数据流
    - **application/x-www-form-urlencoded**：表单提交
    

#### HTTP 请求方法

- **GET**：请求访问已被 URI 识别的资源
    - 提交的内容是 URL 的一部分，长度限制、安全性
- **POST**：与 GET 功能类似，一般用来传输实体的主体，主要目的是提交数据，不是获取响应主体的内容
- **PUT**：与 POST 最大的不同是，PUT 是幂等（不管重复多少次操作，都是实现相同的结果）的，POST 是不幂等的，因此一般创建对象用 POST，更新对象用 PUT
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

- 由于 TCP 提供可靠的、按序的数据传输，他只会交付给应用层按序的数据，所以会有队首阻塞问题，必须要等上一个 HTTP 完整交付之后才能交付下一个
- 建立连接的握手延迟大，TCP 需要握手，对于短连接场景影响大

QUIC 的特性：

- **面向连接和安全**，所有的 QUIC 分组都是加密的。因为 QUIC 用于创建连接状态的握手和加密的握手，因此可以比 TCP + TLS 拥有更快的创建速度。
  ![QUIC建立连接的过程](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-1.png)
- **没有队头阻塞的多路复用**
![TCP队头阻塞](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Blocking.png)
![QUIC没有队头阻塞](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-No-Blocking.png)
- **前向纠错**，会多发一些冗余的包（校验包），虽然产生了多的包，但是减少了重传带来的损失
![前向纠错](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/QUIC-Error-Correcting.png)



## HTTPS

> [How Does HTTPS Work?](https://www.thesslstore.com/blog/how-does-https-work)

### Two Things HTTPS Does

HTTPS 做了两件非常重要的事情：

1. **Authentication**：网站被第三方认证了（Authenticated）、网站的身份也已经被确认过了。比如 https://www.amazon.com 确保你真的在访问 amazon.com。
1. **Encryption**：你与网站之间传输的数据是经过加密的。

可以经常听到 HTTPS 与 **SSL** 和 **TLS**，下面使他们之间大体的关系：

- **Secure Sockets Layer (SSL)** 和 **Transport Layer Security (TLS)** 指的是 HTTPS 用来在服务器与浏览器之间沟通的安全连接类型。
- SSL 是一个相对较老的技术，并且已经被 TLS 取代了。
- 使用上，很多人会将其混用，因此会容易听到 SSL/TLS、SSL/TLS 认证之类的说法。

#### Authentication

> 身份验证（Authentication）保证了我们访问的是正确的网站。

浏览器上的 URL 栏并不能总是保证我们访问了正确的网站，攻击者可以通过 [DNS 下毒](#dns-poisoning)、恶意软件等手段让我们的浏览器访问到错误的网站——即使浏览器的地址栏显示的是正确的 URL。

浏览器通过 SSL/TLS 证书（Certificate）和颁发证书的权威机构（Certificate Authority）来认证访问的 HTTPS 网站，主要有这样几步：

**第一步：网站获取证书**

网站需要获取由第三方可信证书机构颁发的 SSL 证书，这个证书就像网站的护照，他会包含一些信息：

- 网站的 URL
- 一个公钥，该公钥对应着一个该网站才有的私钥
- 证书颁发机构
- 证书过期时间
- 运行网站的合法组织（可选）

网站所有者需要做以下几步来获取证书：

1. 生成一个公钥和一个私钥
2. 通过一些特定的流程向某个证书颁发机构证明他们是这个网站的真正所有者
3. 如果是 OV 和 EV SSL 证书，网站拥有者还需要证明他们是一个真正的、合法的注册组织

网站所有者获取证书之后，会将其安装在 Web 服务器上，然后用户每次通过 https://URL 访问网站时，就会自动向用户提供证书。

**第二步：浏览器索取证书**

当我们访问一个 HTTPS 网站时，网站会给浏览器发送一个 SSL/TLS 证书，可以点一下浏览器地址栏旁边小锁进行查看。

有的证书会展示公司的详情，这类证书被称为 OV (Organization Validation) 或 EV (Extended Validation) 证书，这类证书的公司详情也是被认证过的。

**第三步：浏览器验证证书**

网站出示了证书并不能代表浏览器就能信任他，这些证书还需要通过校验来确保他们的准确性和真实性。更确切地说，浏览器需要确认：

1. 网站的 SSL 证书颁发机构在浏览器的信任机构列表中（浏览器使用证书颁发机构的数字签名来实时确认网站的证书颁发机构）
2. SSL 证书对于正在访问的域名或 URL 来说是有效的
3. SSL 证书当下是有效的，并没有过期或废弃

**总的来说，浏览器会向网站索要并检查证书，就像机场安检员检查护照一样。**

#### Encryption

HTTPS 会对浏览器和网站之间的所有信息进行加密，确保第三方不能读取数据。

### Public Key Infrastructure (PKI)

HTTPS、SSL、TLS 都是在 Public Key Infrastructure (PKI) 的基础上运行的，PKI 就像是他们的引擎。

PKI 由五个部分组成：

1. 数字证书（Digital Certificates），比如 SSL 证书
2. 公钥私钥对（Public Keys & Private Keys）
3. 证书颁发机构（Certificate Authorities）
4. 数字签名（Digital Signatures）
5. 根存储（Root Stores）

#### Public Keys & Private Keys

##### Asymmetric and Symmetric Encryption

对称加密用同样的私钥进行加密和解密，非对称加密则使用一个公钥/私钥对进行加密。

对称加密效率高，但在密钥分发时存在安全隐患，容易被截获；非对称加密安全性高，但性能低下；因此我们可以先用非对称加密传输对称加密的密钥，之后再用对称加密来传输数据。

如果我们有某个网站的证书和公钥，我们就可以发送只有该网站才能解读的信息。这也意味着我们访问的的确是正版网站，因为只有正版网站才能解读我们的信息。

![对称加密](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Encryption.png)

![非对称加密](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/AsymEncryption.png)

#### Digital Signatures

数字签名是用来证明某个应用程序或文档是从某人或某处来的。**如果我们有作者的公钥，我们可以用该公钥来验证他们用私钥签名的东西。**

这个机制非常重要，浏览器用他来确认到底是哪个 CA 颁发了这个 SSL 证书。如果浏览器有 CA 的公钥，那么浏览器就可以确认该证书确实是由该 CA 颁发的。浏览器通过 Root Stores 拿到 CA 的公钥。

![Alice将用自己私钥签名后的信息发送给Bob](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Signature-1.png)

![Bob用Alice的公钥验证信息](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Signature-2.png)



#### Certificate Authorities

如上文所说，CA 是颁发 SSL 证书的可信机构。

![CA用自己的私钥给证书签名](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Certificate-1.png)

#### Digital Certificates

上文已经讲述过了，他就像网站和组织的护照一样。

![Bob用CA的公钥验证数字证书](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Digital-Certificate-2.png)

#### Root Stores

每个设备都有 Root Store 或者 Trust Store，里面保存了他信任的所有 CA 的数字证书和相关的公钥。每个设备都有一个默认的信任 CA 列表，我们也可以手动更改这个列表。

以某个 SSL 证书为例，他可能有多个层次：

```text
DigiCert High Assurance EV Root CA
  - DigCert SHA2 Extended Validation Server CA
    - www.thesslstore.com
```

1. 顶级证书 DigiCert High Assurance EV Root CA 是根证书（Root Certificate），这个证书应该被预装在设备的 Root Sotre 中；
2. 中间的证书被称为 Intermediate Certificate，他被根证书签名（可以用 DigiCert High Assurance EV Root CA 的公钥验证他，这个公钥作为根证书的一部分也被预装在设备中），因此浏览器也可以信任他；
3. 最下面的 www.thesslstore.com 是网站的 SSL 证书。当我们连接的时候，Web 服务器将该证书发送给浏览器，浏览器用此证书中的 DigCert SHA2 Extended Validation Server CA 的签名来验证该证书。

这三个证书被称为“信任链”，浏览器通过这样的方式来确认该 SSL 证书是有效且可信的。

### Enable HTTPS on Your Website

Apache、NGINX、IIS、cPanel、Plesk 等都支持 HTTPS，所以不需要单独安装其他软件，只需要安装 SSL 证书并更新网站设置。

**第一步：获取 SSL/TLS 证书**

> 如果颁发证书的 CA 并没有被浏览器自动标记为信任，访问时就会提示 Your connection is not private。

1. 创建一个 **证书签名请求（Certificate Signing Request, CSR）**，一般可以在 Web Hosting Control Panel（比如 cPanel）上创建该请求，同时也会创建一个公钥私钥对；
2. 将 CSR 提交给 SSL 提供商或 CA；
3. 完成 CA 的验证过程。这会根据证书的类型不同而发生改变，但是至少需要证明你是该域名的拥有者。

**第二步：在网站上安装 SSL 证书**

通常可以在 Web Hosting Control Panel 上安装。

**第三步：更新网站设置使用 HTTPS**

1. 更改 CMS 来对所有的页面使用 https://；
2. 对 http://URL 设置 301 重定向到 https://URL；
3. 确保所有的图片等静态资源、CSS 文件、JavaScript 文件通过 https:// 加载

### The TLS Handshake

当我们访问 HTTPS 网站的时候，浏览器会验证该网站并加密数据。为了做到这些，浏览器和服务器之间会建立一个安全连接，该安全连接的建立过程被称为 SSL/TLS Handshake。

![SSL/TLS Handshake](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SSL_Handshake_10-Steps.gif)

下面的握手过程是基于 TLS 1.2 的，关于 TLS 1.3 及更多信息参考 [Taking a Closer Look at the SSL/TLS Handshake](https://www.thesslstore.com/blog/explaining-ssl-handshake/)。

**Steps 1-2: Hello & Cipher Suites**

服务器和浏览器发送 Hello 并协商使用何种加密协议簇（Cipher Suite），更多关于加密协议簇的信息参考 [Cipher Suites: Ciphers, Algorithms and Negotiating Security Settings](https://www.thesslstore.com/blog/cipher-suites-algorithms-security-settings/)。

**Steps 3-5: Certificate & Key Exchange**

服务器将 SSL 证书、中间证书和相关的公钥发送给浏览器，浏览器根据证书上面的签名和已经信任的根证书来验证 SSL 证书。如果验证失败，连接会被终端并显示错误信息。

如果上述检查都通过了，我们就拥有了一个安全的通信通道：

- 浏览器验证了服务器的 SSL 证书，因此服务器被验证了
- 浏览器拥有了服务器的公钥，浏览器可以加密发送给浏览器的消息了

但是当前的安全连接只是单向的，服务器不能加密发送给浏览器的信息，因为服务器还没有浏览器的公钥。

**Steps 6-10: Setting Up Symmetric Encryption**

对称加密效率是高于浏览器的，因此浏览器和服务器现在会创建一个新的、共享的密钥（该密钥取决于加密协议簇）

当共享的会话密钥创建好之后，TLS Handshake 就结束了。浏览器和服务器会使用该密钥来加密和解密他们之间传输的所有数据。

## Email

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

## Media Stream

- **HTTP 流**：视频只是一个普通的文件，每个文件有一个特定的 URL。接收到视频就进行播放，同时缓存该视频后面部分的帧。
  - 缺陷：所有客户端接收到相同编码的视频
- **DASH (Dynamic Adaptive Stream over HTTP)**：视频编码为几个不同的版本，客户动态地请求来自不同版本且长度为几秒的视频段数据块。每个视频版本存储在服务器中，HTTP 服务器有一个告示文件（manifest file），为每个版本提供一个 URL 和比特率。

## Content Distribution Network (CDN)

> 内容分发网即 CDN (Content Distribution Network)

- CDN 管理分布在多个地理位置的服务器，在他的服务器中存储各种 web 内容的副本，并且将每个用户请求定向到一个将提供最好用户体验的 CDN 位置
- CDN 可以是专用 CDN（private CDN），也可以是第三方 CDN（third-party CDN）

# Transport Layer

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

#### GBN

> GBN 协议也常被称为 **滑动窗口协议（sliding-window protocol）**

![Networking-rdt-6](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-6.jpg)

- 发送方
  - 基序号 base：最早未确认分组的序号
  - 窗口大小 N：当接收到应用层序号大于 base + N 的数据包时，拒绝发送此数据包
  - 累计确认（cumulative acknowledgement）：发送方收到序号为 n 的 ACK 时，认为 n 及 n 以前的都已经被确认了
  - 超时事件：超时后，发送方重传所有已发送但还未确认的分组
- 接收方
  - 丢弃所有失序分组，接收方不需要缓存任何失序分组（包括序号小于 base 的分组）

![Networking-rdt-7](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-7.jpg)

![Networking-rdt-8](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-8.jpg)

#### SR

> 选择重传

![Networking-rdt-9](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-rdt-9.jpg)

- 发送方
  - 与 GBN 一样有窗口大小，会拒绝上层超出当前窗口的包
  - 逐个确认：每个分组有会单独维护自己的 ACK 状态，仅重传那些出错的分组
  - 每个分组有自己的逻辑计时器
- 接收方
  - 缓存失序分组
  - 仅将已收到的连续的分组发送给上层
  - 接收到小于 base 的分组时，也会发送 ACK。因为如果接收方不确认该分组，发送方的窗口可能永远不能向前滑动（SR协议中，发送方和接收方的窗口并不总是一致）
  - 窗口应小于序号空间大小的一半，因为如果窗口太大，会产生歧义而无法判断是旧分组重传还是新分组

### 可靠数据传输机制总结

| 机制         | 用途和说明                                                   |
| ------------ | ------------------------------------------------------------ |
| 校验和       | 检测比特错误                                                 |
| 定时器       | 超时重传，注意接收方可能收到一个分组的多个冗余副本           |
| 序号         | 检测出丢失副本和冗余副本                                     |
| 确认         | 告诉发送方一个或多个分组已经收到，有逐个确认、累计确认等方式 |
| 否定确认     | 告诉发送方某个分组未被正确接收                               |
| 窗口、流水线 | 与停等相比，有更高的发送方利用率                             |

## TCP

> TCP 面向连接、可靠、面向字节流；UDP 面向无连接

1. **面向连接**。TCP 需要三次握手建立连接，UDP 则不需要
2. **可靠**。TCP **有状态**，会记录哪些数据发送了，哪些数据对方接受了，哪些数据丢失了，不允许差错；TCP **可控制**，会根据丢包的情况或者状态，调整自己的行为，比如控制速度或者重发
3. TCP 提供的是 **全双工服务（full-duplex service）**：建立连接后，A 的数据流向 B 的同时，B 也可以流向 A，A 既是发送方也是接收方
4. **面向字节流**。UDP 的数据传输 **是面向数据报** 的，TCP 为了维护状态，将 IP 包变成了字节流。所谓 **面向数据报** 是指，应用层交给 UDP 多长的 **报文**，UDP 就照样发送，一次发送一个报文；而面向字节流是指，TCP 虽然一次接收应用层来的一个数据块（大小不等），但 TCP 把应用程序看成一连串无结构的字节流。TCP 有一个缓冲，当应用程序传送的数据块太长，TCP 就会把它划分得短一些再发送；如果应用程序一次只发送一个字节，TCP 也可以等待积累足够多的字节后再构成报文段发送出去。

### 报文结构

- TCP 报文可以看成 报文头 + 数据
- **最大报文段长度（Maximum Segment Size）**：TCP 发送一个大文件时，会将一个文件划分为长度为 MSS 的若干块；发送一个小数据块的时候，每个报文段长度将会远小于 MSS
- 下图是报文头的结构

![TCP报文头](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Header.png)

#### 序列号

> SN (Sequence Number)，长度为 4 Bytes

![Networking-TCP-segment-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-TCP-segment-2.jpg)

- TCP 的序号是建立在字节流上面的，而不是报文段之上
  - 假设发送一个文件有 500 000 Bytes，MSS 为 1000 Bytes
  - TCP 将会为每一个字节隐式地编号，假设数据流首字节编号为 0
  - TCP 将会为该数据流构造 500 个报文段，第一个报文序号为 0，第二个报文序号为 1000，第三个报文序号为 2000

##### 初始序列号

即 Initial Sequence Number，在三次握手的过程中，双方会通过 SYN 报文来交换彼此的 ISN，ISN 会动态增长，每 4ms 就加 1，溢出则回到 0。动态增长的 ISN 增加了猜测 ISN 的难度

#### 确认号

> ACK (Acknowledgement Number)，长度为 4 Bytes

- 告知对方下一个期望接收的序列号
- 小于 ACK 的所有字节已经全部接收到（累计确认）
- TCP 并没有规定对失序到达的报文的处理，而是交给 TCP 编程人员去处理，实践中一般收方会缓存失序字节

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

### 超时重传

> 在间隔一段时间没有等到这个数据包回复时，将会重传这个数据包

#### 超时重传时间的计算

##### 往返时间估计

> 报文段的样本 RTT（SampleRTT）是指某段报文被发出到确认被收到之间的时间

- TCP 仅为传输一次的报文段测量 SampleRTT，不会测量重传报文段的 SampleRTT
- TCP 会维持一个 SampleRTT 的均值（称为 EstimatedRTT）

```text
EstimatedRTT = α * SampleRTT + (1 - α) * EstimatedRTT
```

α 即为 **平滑因子**，建议值是 0.125

##### 测量 RTT 变化

> RTT 变化定义为 DevRTT

```text
DevRTT = (1 - β) * DevRTT + β * |SampleRTT - EstimatedRTT|
```

β 的建议值是 0.25

##### 设置和管理超时时间

```text
TimeoutInterval = µ * EstimatedRTT + ∂ * DevRTT 
```

µ 建议值取 1, ∂ 建议值取 4

#### 超时重传策略

下面是大多数 TCP 实现中做的一些修改，有的在协议中只是提供了一些建议。

##### 超时间隔加倍

>  每次超时事件发生时，就将超时间隔设置为之前的两倍，而不是使用 EstimatedRTT 和 DevRTT 推算出来的值

这种思路提供了一个简单的拥塞控制，防止重传导致的加重拥塞

##### 快速重传

1. 如果接收方收到了比预期序号大的失序报文段，将立即发送一个冗余 ACK（再次确认之前的报文段已经到达）
2. 接收方收到 3 个冗余 ACK 后，就执行快速重传（fast retransmit），即使该报文段没有超时，也进行重传

##### GBN + SR

- TCP 用了累计确认，看起来像 GBN
- TCP 会缓存失序报文段，并不会重传所有未确认报文段，看起来像 SR
- 可以将 TCP 看做 GBN 和 SR 的混合体

### 流量控制

> 流量控制（flow-control service）是一个速度匹配服务，协调发送方的发送速率与接收方的接收速率，消除发送方使接收方缓存溢出的可能性

![Networking-TCP-3](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-TCP-3.jpg)

- **发送方** 维护一个 **接收窗口（receive window）** 变量（即 **rwnd**）该变量指示了接收方剩余的缓存空间
- **接收方** 把当前 rwnd 值放入报文段的接收窗口字段中，通知 **发送方** 该连接的缓存中还有多少剩余空间
- 当 **接收方** 的 rwnd 为 0 时，**发送方** 继续发送只有一个字节数据的报文段。接收方会确认这些报文段，否则发送方将不会知道何时缓存开始清空
- UDP 不提供流量控制，缓存溢出后报文段将会丢失

## TCP 连接管理

### 三次握手

> 三次握手是需要确认双方的 **发送能力** 和 **接收能力**

![TCP三次握手](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Handshake.png)

> **三次握手的过程中可以携带数据吗？**
>
> 前两次不能，第三次可以，此时客户端已经处于 ESTABLISHED  状态，确认了服务器发送、接受信息的能力正常

### 四次挥手

![TCP四次挥手](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/TCP-Wavehand.png)

服务端在接收到 FIN, 往往不会立即返回 FIN, 必须等到服务端所有的报文都发送完毕了，才能发 FIN。因此先发一个 ACK 表示已经收到客户端的 FIN，延迟一段时间才发 FIN。这就造成了四次挥手。

### SYN 泛洪攻击与 Cookie

三次握手之前，在服务器状态从 CLOSED 变为 LISTEN 的时候，还在内部创建了两个队列：**半连接队列** 和 **全连接队列**

#### 半连接队列

又称 **SYN 队列**，当客户端发送 SYN 到服务端，服务端收到后回复 ACK 和 SYN，状态由 LISTEN 变为 SYN_RCVD，此时这个连接就被推入了 **半连接队列**

#### 全连接队列

又称 **ACCEPTED 队列**，当客户端返回 ACK，服务端接收后，三次握手完成，连接等待被某个具体的应用取走，在被取走之前，会被推入 **全连接队列**

#### SYN 泛洪攻击

属于典型的 Dos/DDos 攻击，用客户端在短时间内伪造大量不存在的 IP 地址，疯狂发送 SYN 而不发送 ACK，不完成三次握手，使得服务器：

1. 处理大量 SYN 包并返回 ACK，使得 **半连接队列爆满**
2. 由于是不存在的 IP，服务端长时间 **收不到客户端的 ACK**，就会疯狂重发数据，直到耗尽服务器资源

##### 如何应对 SYN 泛洪攻击？

1. 增加半连接队列容量
2. 减少 SYN + ACK 的重试次数
3. 利用 SYN Cookie 技术，在服务端接收到 SYN 后不立即分配链接资源，而是根据 SYN 计算出一个 Cookie，连通第二次握手一同发给客户端，客户端回复 ACK 的时候带上 Cookie，服务端验证合法后才分配链接资源
   - TCP Fast Open：后续再进行三次握手时，客户端可以直接将之前缓存的 Cookie、SYN 和 HTTP 请求发送给服务端，如果合法就正常返回 **SYN + ACK**，同时可以给客户端发 **HTTP 响应**

### 拥塞控制

> 在网络拥塞的情况下，分组重传会作为征兆出现，但并不能解决网络拥塞本身

#### 拥塞控制原理

成因：网络中有很多主机，而物理链路的容量是有限的，网络会随着主机发送速率的增加而逐渐变得拥塞，就像堵车一样

代价：

1. 即使在路由器缓存无限的情况下，分组的到达速率接近链路容量时，分组将经历巨大的排队时延
2. 当路由器缓存有限时，发送方必须执行重传来补偿缓存溢出导致的丢失分组
3. 当分组实际上并没有丢失，只是排队比较久时，发送方的提前的重传将占用额外的资源
4. 当一个分组沿一条路径被丢弃时，每一个上游路由器之前做的所有努力都白费了

控制方法：

- 端到端的拥塞控制。网络层没有提供拥塞控制的方法，因此端系统需要通过网络行为（比如分组丢失与时延）来进行判断。TCP 采用这种方法，因为 IP 层不会反馈网络信息
- 网络辅助的拥塞控制。路由器会向发送方提供网络的拥塞状态。最新的 IP 和 TCP 也能选择性地实现网络辅助的拥塞控制

#### 端到端的拥塞控制

##### 拥塞检测

出现丢包时，即可能出现了拥塞：

- 超时
- 收到来自接收方的 3 个冗余 ACK

##### 拥塞窗口

除了接收窗口 rwnd 以外，发送端另外还需维护 **拥塞窗口（congestion window）** 变量（即 **cwnd**），限制其发送速率

发送方中未被确认的数据量不会超过 rwnd 与 cwnd 的最小值

```text
LastByteSent - LastByteAcked = min(rwnd, cwnd)
```

通过调整 cwnd 就可以调整发送速率，通过用拥塞控制算法可以调整 cwnd

##### 拥塞控制算法

拥塞控制算法分为 **慢启动、拥塞避免、快速恢复** 三部分，其中前两个部分是 TCP 的强制部分，快速恢复是推荐部分

![Networking-TCP-4](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Networking-TCP-4.jpg)

###### 慢启动

> 刚创建连接时，TCP 进入慢启动模式，并迅速适应可用的带宽数量

- 双方初始化自己 **拥塞窗口（cwnd）** 的大小，为 1 MSS
- 发送方发送出第 1 个报文段并等待确认，当报文段首次被确认时，cwnd 增加 1 MSS 变为 2 MSS
- 然后发出 2 个报文段，当他们都被确认时，cwnd 增加 2 MSS 变为 4 MSS
- 然后发出 4 个报文段，当他们都被确认时，cwnd 增加 4 MSS 变为 8 MSS……（每确认一个就增加 1 MSS）

结束指数增长的条件：

- 拥塞发生（丢包事件，包括超时和 3 个冗余 ACK）
  - TCP 发送方将 cwnd 设置为 1 并重新开始慢启动
  - 同时，发送方还有一个 **慢启动阈值（ssthresh）** 的变量，设置为 cwnd / 2，即当检测到拥塞时将 ssthresh 设置为拥塞窗口值的一半
- 当之后 cwnd 等于 ssthresh 时，结束慢启动并转移到 **拥塞避免模式**

###### 拥塞避免

- 进入拥塞避免状态时，cwnd 值大约是上次遇到拥塞时的一半
- 此时每个 RTT 只将 cwnd 的值增加 1 MSS
- 当出现超时，与慢启动状态时相同，ssthresh 的值更新为 cwnd / 2，将 cwnd 重置为 1，然后迁移到 **慢启动状态**
- 当出现 3 个冗余 ACK 时，网络继续发送报文段，将 cwnd 值减半（因为已收到的 3 个冗余 ACK，也要加上 3 个 MSS），将 ssthresh 更新为 cwnd 值的一半，进入 **快速恢复状态**

###### 快速恢复

- 每当收到冗余 ACK，cwnd 的值增加 1 MSS
- 最终，当对丢失报文段的一个 ACK 到达时，进入拥塞避免状态
- 当出现超时，与慢启动、拥塞避免相同，ssthresh 的值更新为 cwnd / 2，cwnd 重置为 1，然后迁移到 **慢启动状态**

#### 网络辅助拥塞控制

##### 明确拥塞通告（ECN）

> 明确拥塞通告（Explicit Congestion Notification, ECN）是一种网络辅助的拥塞控制形式，涉及 TCP 和 IP

- 在网络层，位于 IP 的服务类型字段中有 2 bits 用于 ECN
- 当接收主机得到 ECN 拥塞指示时，发送给发送方的 ACK 包中的 ECE（明确拥塞通告回显）置为 1
- 发送方收到后会减半 cwnd，并在下一个报文段中对 CWR（拥塞窗口缩减）置为 1

##### 基于时延的拥塞控制

> 在丢包之前主动检测拥塞

#### 公平性

> 如果每条连接连接得到相同分量的链路带宽，则认为该拥塞控制机制是公平的

- 理想化条件下，TCP 是公平的，尽管 cwnd 不同
- 实践中，一般具有较小 RTT 的连接能够在链路空闲时更快抢到更多的可用带宽，因此拥有更高的吞吐量
- 由于 UDP 没有拥塞控制，所以 UDP 可以抢占大量资源而拒绝与 TCP 合作
- 此外可以给一个应用建立多条 TCP 连接以破坏公平性

## 其他性能优化

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

### TCP 的 keep-alive

由于 TCP 不是一个轮询的协议，他无法获知对端连接失效的情况（比如对端网络故障或宕机），**keep-alive** 则是用于探测对端的连接是否失效，但是因为探测的时间比较长，所以一般不怎么用

# Network Layer

# Data Link Layer

## Framing

数据链路层会将网络层传下来的分组添加首部和尾部，用于标记帧的开始和结束。

帧通过首部和尾部彼此区分开来，如果真的数据部分包含有首部和尾部相同的内容，那么帧的分界就会被错误地判定，因此需要在数据部分出现的相同内容前面插入转义字符。用户将察觉不到转义字符的存在，该过程又被称为对转义字符的 *透明传输*。

## Error Detection

数据链路层目前通常使用循环冗余检验（Cyclic Redundancy Check, CRC）来检查比特差错。

与校验和不同，CRC 是将两个字节数据流采用二进制除法（而不是加法）来实现的。

## Link

有两种类型的网咯链路：**点对点链路（Point-to-Point Link）**和 **广播链路（Broadcast Link）**。

**广播链路** 是一对多通信，所有节点都在同一个广播信道上发送数据。如果所有节点同时接到多个帧，那么称传输的帧在接收方发生了 **碰撞（Collide）**，此时所有碰撞涉及的帧都丢失了。因此需要专门的控制方法来避免发生碰撞。

可以使用 **多路访问协议（Multiple Access Protocol）**来进行协调，比如 CSMA/CD。

**点对点链路** 不会发生碰撞，因此可以使用较为简单的 **PPP (Point-to-Point Protocol)** 进行控制。

### Multiple Access Protocol

多路访问协议可以分为 **信道划分协议（Channel Partitioning Protocol）**、**随机接入协议（Random Access Protocol）**和 **轮流协议（Taking-Turns Protocol）**。



#### Channel Partitioning Protocol



#### Random Access Protocol

#### Taking-Turns Protocol

# Physical Layer

## Communication Channels

有三种通信方式：

- 单工（Simplex）：单向传输
- 半双工（Half-Duplex）：双向交替传输
- 全双工（Full-Duplex）：双向同时传输

## Modulation

> 带通调制（Modulation）将离散的数字信号转换为连续的模拟信号。

# Network Security

## DNS Poisoning

> 简单来说，DNS 服务器（缓存）被攻击了，导致浏览器得到了一个错误的 IP 地址

发现 DNS 中毒攻击：

- 网络流量原因不明突然下降
- 以客户的身份（可以换一台电脑、用 VPN 改变位置等）尝试访问网站，看是不是被重定向到奇怪的位置

Further Reading：

[DNS Poisoning Attacks: A Guide for Website Admins](https://www.thesslstore.com/blog/dns-poisoning-attacks-a-guide-for-website-admins/)

XSS

CSRF
