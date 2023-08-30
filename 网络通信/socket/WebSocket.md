## 一、传统双工通信方案

##### 1、传统轮询(Traditional Polling)
既客户端利用ajax定时轮询，来达到不停更新数据的目的。

缺陷：程序在每次请求时都会新建一个HTTP请求，然而并不是每次都能返回所需的新数据。当同时发起的请求达到一定数目时，会对服务器造成较大负担。

##### 2、长轮询(Long Polling)
为了解决传统轮询的缺陷，出现了长轮询。

长轮询的基本思想是在每次客户端发出请求后，服务器检查上次返回的数据与此次请求时的数据之间是否有更新，如果有更新则返回新数据并结束此次连接，否则服务器hold住(使用while循环hold住程序)此次连接，直到有新数据时再返回相应。而这种长时间的保持连接可以通过设置一个较大的 HTTP timeout` 实现。

缺陷：每次连接的保持是以消耗服务器资源为代价的。尤其对于Apache+PHP 服务器，由于有默认的 worker threads 数目的限制，当长连接较多时，服务器便无法对新请求进行相应。

##### 3、服务器发送事件(Server-Sent Event)
服务器发送事件（以下简称SSE）是HTML 5规范的一个组成部分，可以实现服务器到客户端的**单向数据通信**。

缺陷：服务端依然是用while挂起程序。SSE只支持服务器到客户端单向的事件推送，而且所有版本的IE（包括到目前为止的Microsoft Edge）都不支持SSE。

于是就引出今天的主角：WebSocket。

## 二、WebSocket
### 1、简介
WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。

WebSocket同样是HTML 5规范的组成部分之一，现标准版本为 RFC 6455。WebSocket 相较于上述几种连接方式，实现原理较为复杂，用一句话概括就是：客户端向 WebSocket 服务器通知（notify）一个带有所有 接收者ID（recipients IDs） 的事件（event），服务器接收后立即通知所有活跃的（active）客户端，只有ID在接收者ID序列中的客户端才会处理这个事件。由于 **WebSocket 本身是基于TCP协议的**，所以在服务器端我们可以采用构建 TCP Socket 服务器的方式来构建WebSocket 服务器。

**WebSocket 是一种全新的协议。它将 TCP 的 Socket（套接字）应用在了web page上，从而使通信双方建立起一个保持在活动状态连接通道，是全双工协议（双方同时进行双向通信）**。

其实是这样的，WebSocket 协议是借用 HTTP协议 的 101 switch protocol 来达到协议转换的，**从HTTP协议切换成WebSocket通信协议（即websocke是通过http upgrade而来的）**。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。其他特点包括：

* 使得客户端和服务器之间的数据交换变得更加简单，**允许服务端主动向客户端推送数据**
* 在WebSocket API中，**浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输**
* 建立在 TCP 协议之上，服务器端的实现比较容易。
* 与 HTTP 协议有着良好的兼容性。默认端口也是 80 和 443 ，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
* 数据格式比较轻量，性能开销小，通信高效。
* 可以发送文本，也可以发送二进制数据。
* 没有同源限制，客户端可以与任意服务器通信。
* 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

### 2、协议
WebSocket协议被设计来取代现有的使用HTTP作为传输层的双向通信技术，并受益于现有的基础设施（代理、过滤、身份验证）。

**协议有两部分：握手和数据传输。**

##### 握手

> 客户端：申请协议升级，客户端发起协议升级请求。可以看到，采用的是标准的 HTTP 报文格式，且只支持GET方法。

来自客户端的握手看起来像如下形式：
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

重点请求首部意义如下：
* Connection: Upgrade：表示要申请升级协议
* Upgrade: websocket：表示要升级到 websocket 协议。
* Sec-WebSocket-Version: 13：表示 websocket 的版本。如果服务端不支持该版本，需要返回一个 Sec-WebSocket-Versionheader ，里面包含服务端支持的版本号。
* Sec-WebSocket-Key：与后面服务端响应首部的 Sec-WebSocket-Accept 是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。

> 服务端：响应协议升级。服务端返回内容如下，状态代码101表示协议切换。到此完成协议升级，后续的数据交互都按照新的协议来。

来自服务器的握手看起来像如下形式：
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

重点请求首部意义如下：
* Sec-WebSocket-Accept 根据客户端请求首部的 Sec-WebSocket-Key 计算出来。
* Upgrade: websocket 表示允许升级为websocket

##### 数据传递
一旦 WebSocket 客户端、服务端建立连接后，后续的操作都是基于数据帧的传递。

WebSocket 客户端、服务端通信的最小单位是 **帧（frame）**，由 1 个或多个帧组成一条完整的 消息（message）。

* 发送端：将消息切割成多个帧，并发送给服务端；
* 接收端：接收消息帧，并将关联的帧重新组装成完整的消息；

### 3、连接保持 + 心跳
WebSocket 为了保持客户端、服务端的实时双向通信，需要确保客户端、服务端之间的 TCP 通道保持连接没有断开。然而，对于长时间没有数据往来的连接，如果依旧长时间保持着，可能会浪费包括的连接资源。

但不排除有些场景，客户端、服务端虽然长时间没有数据往来，但仍需要保持连接。这个时候，可以采用心跳来实现。

* 发送方 ->接收方：ping
* 接收方 ->发送方：pong

### 4、关闭连接
一旦发送或接收到一个Close控制帧，这就是说，WebSocket 关闭阶段握手已启动，且 WebSocket 连接处于 CLOSING 状态。

当底层TCP连接已关闭，这就是说 WebSocket连接已关闭 且 WebSocket 连接处于 CLOSED 状态。 如果 TCP 连接在 WebSocket 关闭阶段已经完成后被关闭，WebSocket连接被说成已经 完全地 关闭了。

如果WebSocket连接不能被建立，这就是说，WebSocket连接关闭，但不是 完全的 。

### 5、代码简单实现
客户端
```js
 <script type="text/javascript">
        var websocket = new WebSocket("ws://echo.websocket.org/");
        // 引入websocket
        websocket.onopen = function(){
            console.log('websocket open');
            document.getElementById("recv").innerHTML = "Connected";
        }
        // 结束websocket
        websocket.onclose = function(){
            console.log('websocket close');
        }
        // 接受到信息
        websocket.onmessage = function(e){
            console.log(e.data);
            document.getElementById("recv").innerHTML = e.data;
        }
        // 点击发送webscoket
        document.getElementById("sendBtn").onclick = function(){
            var txt = document.getElementById("sendTxt").value;
            websocket.send(txt);
        }
    </script>
```
服务端：正常socket编程。

### 6、相关疑问
1. 和TCP、HTTP协议的关系

    WebSocket 是基于 TCP 的独立的协议。它与 HTTP 唯一的关系是它的握手是由 HTTP 服务器解释为一个 Upgrade 请求。
    
    WebSocket协议试图在现有的 HTTP 基础设施上下文中解决现有的双向HTTP技术目标；同样，它被设计工作在HTTP端口80和443，也支持HTTP代理和中间件，
    
    HTTP服务器需要发送一个“Upgrade”请求，即101 Switching Protocol到HTTP服务器，然后由服务器进行协议转换。
    
2. Sec-WebSocket-Key/Accept 的作用
    前面提到了，Sec-WebSocket-Key/Sec-WebSocket-Accept 在主要作用在于提供基础的防护，减少恶意连接、意外连接。
    
    作用大致归纳如下：
    
    避免服务端收到非法的 websocket 连接（比如 http 客户端不小心请求连接 websocket 服务，此时服务端可以直接拒绝连接）
    
    确保服务端理解 websocket 连接。因为 ws 握手阶段采用的是 http 协议，因此可能 ws 连接是被一个 http 服务器处理并返回的，此时客户端可以通过 Sec-WebSocket-Key 来确保服务端认识 ws 协议。（并非百分百保险，比如总是存在那么些无聊的 http 服务器，光处理 Sec-WebSocket-Key，但并没有实现 ws 协议。。。）
    
    **用浏览器里发起 ajax 请求，设置 header 时，Sec-WebSocket-Key 以及其他相关的 header 是被禁止的。这样可以避免客户端发送 ajax 请求时，意外请求协议升级（websocket upgrade）**
    
    可以防止反向代理（不理解 ws 协议）返回错误的数据。比如反向代理前后收到两次 ws 连接的升级请求，反向代理把第一次请求的返回给 cache 住，然后第二次请求到来时直接把 cache 住的请求给返回（无意义的返回）。
    
    **Sec-WebSocket-Key 主要目的并不是确保数据的安全性，因为 Sec-WebSocket-Key、Sec-WebSocket-Accept 的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）**。
    
    
