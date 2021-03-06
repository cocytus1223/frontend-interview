# 1. 网络类

## 1.1. 一个页面从输入 url 到页面加载显示完成，这个过程发生了什么？

1. 在浏览器地址栏输入 URL

2. 浏览器查看**缓存**，如果请求资源在缓存中并且新鲜，跳转到转码步骤

   - 如果资源未缓存，发起新请求
   - 如果已缓存，检验是否足够新鲜，足够新鲜直接提供给客户端，否则与服务器进行验证。
   - 检验新鲜通常有两个 HTTP 头进行控制`Expires`和 `Cache-Control`：
     - HTTP1.0 提供 `Expires`，值为一个绝对时间表示缓存新鲜日期
     - HTTP1.1 增加了 `Cache-Control: max-age=`,值为以秒为单位的最大新鲜时间

3. 浏览器**解析 URL**获取协议，主机，端口，path

4. 浏览器**组装一个 HTTP（GET）请求报文**

5. 浏览器**获取主机 ip 地址**，过程如下：

   1. 浏览器缓存
   2. 本机缓存
   3. hosts 文件
   4. 路由器缓存
   5. ISP DNS 缓存
   6. DNS 递归查询（可能存在负载均衡导致每次 IP 不一样）

6. 打开一个 socket 与目标 IP 地址，端口建立 TCP 链接，三次握手如下：

   1. 客户端发送一个 TCP 的**SYN=1，Seq=X**的包到服务器端口，客户端进入 SYN_SEND 状态。
   2. 服务器发回**SYN=1， ACK=X+1， Seq=Y**的响应包，服务器端进入 SYN_RCVD 状态。
   3. 客户端发送**ACK=Y+1， Seq=Z**

7. TCP 链接建立后**发送 HTTP 请求**

8. 服务器接受请求并解析，将请求转发到服务程序，如虚拟主机使用 HTTP Host 头部判断请求的服务程序

9. 服务器检查**HTTP 请求头是否包含缓存验证信息**如果验证缓存新鲜，返回**304**等对应状态码

10. 处理程序读取完整请求并准备 HTTP 响应，可能需要查询数据库等操作

11. 服务器将**响应报文通过 TCP 连接发送回浏览器**

12. 浏览器接收 HTTP 响应，然后根据情况选择

    关闭 TCP 连接或者保留重用，关闭 TCP 连接的四次握手如下：

    1. 主动方发送**Fin=1， Ack=Z， Seq= X**报文
    2. 被动方发送**ACK=X+1， Seq=Z**报文
    3. 被动方发送**Fin=1， ACK=X， Seq=Y**报文
    4. 主动方发送**ACK=Y， Seq=X**报文

13. 浏览器检查响应状态吗：是否为 1XX，3XX， 4XX， 5XX，这些情况处理与 2XX 不同

14. 如果资源可缓存，**进行缓存**

15. 对响应进行**解码**（例如 gzip 压缩）

16. 根据资源类型决定如何处理（假设资源为 HTML 文档）

17. **解析 HTML 文档，构件 DOM 树，下载资源，构造 CSSOM 树，执行 js 脚本**，这些操作没有严格的先后顺序，以下分别解释

18. 构建 DOM 树：

    1. **Tokenizing**：根据 HTML 规范将字符流解析为标记
    2. **Lexing**：词法分析将标记转换为对象并定义属性和规则
    3. **DOM construction**：根据 HTML 标记关系将对象组成 DOM 树

19. 解析过程中遇到图片、样式表、js 文件，**启动下载**

20. 构建 CSSOM 树：

    1. **Tokenizing**：字符流转换为标记流
    2. **Node**：根据标记创建节点
    3. **CSSOM**：节点创建 CSSOM 树

21. 根据 DOM 树和 CSSOM 树构建渲染树:

    1. 从 DOM 树的根节点遍历所有**可见节点**，不可见节点包括：1）`script`,`meta`这样本身不可见的标签。2)被 css 隐藏的节点，如`display: none`
    2. 对每一个可见节点，找到恰当的 CSSOM 规则并应用
    3. 发布可视节点的内容和计算样式

22. js 解析如下：

    1. 浏览器创建 Document 对象并解析 HTML，将解析到的元素和文本节点添加到文档中，此时**document.readystate 为 loading**
    2. HTML 解析器遇到**没有 async 和 defer 的 script 时**，将他们添加到文档中，然后执行行内或外部脚本。这些脚本会同步执行，并且在脚本下载和执行时解析器会暂停。这样就可以用 document.write()把文本插入到输入流中。**同步脚本经常简单定义函数和注册事件处理程序，他们可以遍历和操作 script 和他们之前的文档内容**
    3. 当解析器遇到设置了**async**属性的 script 时，开始下载脚本并继续解析文档。脚本会在它**下载完成后尽快执行**，但是**解析器不会停下来等它下载**。异步脚本**禁止使用 document.write()**，它们可以访问自己 script 和之前的文档元素
    4. 当文档完成解析，document.readState 变成 interactive
    5. 所有**defer**脚本会**按照在文档出现的顺序执行**，延迟脚本**能访问完整文档树**，禁止使用 document.write()
    6. 浏览器**在 Document 对象上触发 DOMContentLoaded 事件**
    7. 此时文档完全解析完成，浏览器可能还在等待如图片等内容加载，等这些**内容完成载入并且所有异步脚本完成载入和执行**，document.readState 变为 complete,window 触发 load 事件

23. **显示页面**（HTML 解析过程中会逐步显示页面）

## 1.2. http 的常用的状态码遇到过那些？分别代表什么意思？

- 1xx：指示信息——表示请求已接受，继续处理
- 2xx：成功——表示请求已被成功接收
- 3xx：重定向——要完成请求必须进行更进一步的操作
- 4xx：客户端错误——请求有语法错误或请求无法实现
- 5xx：服务器错误——服务器未能实现合法的请求

200 OK：客户端请求成功
206 Partial Content：客户发送了一个带有 Range 头的 GET 请求，服务器完成了它
301 Moved Permanently：永久重定向，所请求的页面已经转移到新的 url
302 Found：临时重定向，所请求的页面已经临时转移至新的 url
304 Not MOdified：缓存，客户端有缓冲的文档并发出了一个条件性的请求，服务器告诉客户，原来缓冲的文档还可以继续使用
400 Bad Request：客户端有语法错误，不能被服务器所理解
401 Unauthorized：请求未经授权，这个状态码必须和 WWW-Authenticate 报头域一起使用，当前请求需要用户验证
403 Forbidden：对被请求页面的访问被禁止，服务器已经得到请求，但是拒绝执行
404 Not Found：请求资源不存在
500 Internal Server Error：服务器发生不可预期的错误原来缓冲的文档还可以继续使用
503 Server Unavailable：请求未完成，服务器临时过载或宕机，一段时间后可能恢复正常

补充：400 状态码，请求无效

- 产生原因：

  - 前端提交数据的字段名称和字段类型与后台的实体没有保持一致
  - 前端提交到后台的数据应该是 json 字符串类型，但是前端没有将对象 JSON.stringify 转化成字符串。

- 解决方法：

  - 对照字段的名称，保持一致性
  - 将 obj 对象通过 JSON.stringify 实现序列化

## 1.3. get 和 post 的区别

1. GET 在浏览器回退时是无害的，而 POST 会再次提交请求 ☆
2. GET 产生的 URL 地址是可以被收藏的，而 POST 不可能
3. GET 请求会被浏览器自动缓存，而 POST 不可以，除非手动设置 ☆
4. GET 请求只能进行 URL 编码，而 POST 支持多种编码方式
5. GET 请求参数会被完整保留在浏览器历史记录中，而 POST 中的参数不会被保留 ☆
6. GET 请求在 URL 中传送的参数是有长度限制的（2KB，不同浏览器不同，URL 拼接的时候要注意长度），而 POST 没有限制 ☆
7. 对参数的数据类型，GET 只接受 ASCII 字符，而 POST 没有限制
8. GET 比 POST 更不安全，因为参数直接暴露在 URL 上，所以不能用来传递敏感信息
9. GET 参数通过 URL 传递，POST 是放在 Request body 中 ☆

## 1.4. get 请求传参长度的误区

误区：我们经常说 get 请求参数的大小存在限制，而 post 请求的参数大小是无限制的。

实际上 HTTP 协议从未规定 GET/POST 的请求长度限制是多少。对 get 请求参数的限制是来源与浏览器或 web 服务器，浏览器或 web 服务器限制了 url 的长度。为了明确这个概念，我们必须再次强调下面几点:

- HTTP 协议 未规定 GET 和 POST 的长度限制
- GET 的最大长度显示是因为 浏览器和 web 服务器限制了 URI 的长度
- 不同的浏览器和 WEB 服务器，限制的最大长度不一样
- 要支持 IE，则最大长度为 2083byte，若只支持 Chrome，则最大长度 8182byte

## 1.5. 补充 get 和 post 请求在缓存方面的区别

- get 请求类似于查找的过程，用户获取数据，可以不用每次都与数据库连接，所以可以使用缓存。
- post 不同，post 做的一般是修改和删除的工作，所以必须与数据库交互，所以不能使用缓存。因此 get 请求适合于请求缓存。

## 1.6. WebSocket 的实现和应用

### 1.6.1. 什么是 WebSocket？

WebSocket 是 HTML5 中的协议，支持持久连续，http 协议不支持持久性连接。Http1.0 和 HTTP1.1 都不支持持久性的链接，HTTP1.1 中的 keep-alive，将多个 http 请求合并为 1 个

### 1.6.2. WebSocket 是什么样的协议，具体有什么优点？

HTTP 的生命周期通过 Request 来界定，也就是 Request 一个 Response，那么在 Http1.0 协议中，这次 Http 请求就结束了。在 Http1.1 中进行了改进，是的有一个 connection：Keep-alive，也就是说，在一个 Http 连接中，可以发送多个 Request，接收多个 Response。但是必须记住，在 Http 中一个 Request 只能对应有一个 Response，而且这个 Response 是被动的，不能主动发起。
WebSocket 是基于 Http 协议的，或者说借用了 Http 协议来完成一部分握手，在握手阶段与 Http 是相同的。我们来看一个 websocket 握手协议的实现，基本是 2 个属性，upgrade，connection。
基本请求如下：

```html
GET /chat HTTP/1.1 Host: server.example.com Upgrade: websocket Connection:
Upgrade Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== Sec-WebSocket-Protocol:
chat, superchat Sec-WebSocket-Version: 13 Origin: http://example.com
```

多了下面 2 个属性：

```html
Upgrade:webSocket Connection:Upgrade 告诉服务器发送的是websocket
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== Sec-WebSocket-Protocol: chat,
superchat Sec-WebSocket-Version: 13
```

## http2.0

简要概括：http2.0 是基于 1999 年发布的 http1.0 之后的首次更新。

- 提升访问速度（可以对于，请求资源所需时间更少，访问速度更快，相比 http1.0）
- 允许多路复用：多路复用允许同时通过单一的 HTTP/2 连接发送多重请求-响应信息。改善了：在 http1.1 中，浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制（连接数量），超过限制会被阻塞。
- 二进制分帧：HTTP2.0 会将所有的传输信息分割为更小的信息或者帧，并对他们进行二进制编码
- 首部压缩
- 服务器端推送

## fetch 发送 2 次请求的原因

fetch 发送 post 请求的时候，总是发送 2 次，第一次状态码是 204，第二次才成功？

原因很简单，因为用 fetch 的 post 请求的时候，导致 fetch 第一次发送了一个 Options 请求，询问服务器是否支持修改的请求头，如果服务器支持，则在第二次中发送真正的请求。
