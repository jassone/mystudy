## TCP 长连接和短连接
## 短连接
模拟一种TCP短连接的情况:

1. client 向 server 发起连接请求
2. server 接到请求，双方建立连接
3. client 向 server 发送消息
4. server 回应 client
5. 一次读写完成，此时双方任何一个都可以发起 close 操作
6. 在步骤5中，一般都是 client 先发起 close 操作。当然也不排除有特殊的情况。

从上面的描述看，短连接一般只会在 client/server 间传递一次读写操作！
## 二、TCP 长连接
长连接的情况:
1. client 向 server 发起连接
2. server 接到请求，双方建立连接
3. client 向 server 发送消息
4. server 回应 client
5. 一次读写完成，连接不关闭
6. 后续读写操作...

长时间操作之后client或server发起关闭请求

### 1、TCP长连接保持的两种办法：
* 应用层面的心跳机制
    自定义心跳消息头.，一般客户端主动发送到服务端，服务器接收后进行回应(也可以不回应)，以便能够侦测连接是否异常断开。

* **TCP协议自带的保活功能：TCP Keep Alive**

##### 何为Keep Alive

TCP长连接(Keep Alive) 是一种保持 TCP 连接的机制。当一个 TCP 连接建立之后，启用 TCP Keep Alive 的一端便会启动一个计时器，当这个计时器到达 0 之后，一个 TCP 探测包便会被发出。这个 TCP 探测包是一个纯 ACK 包，但是其 Seq 与上一个包是重复的。

打个比喻：
TCP 连接两端好比两个人，这两个人之间保持通信往来（建立 TCP 连接）。如果他俩经常通信（经常发送 TCP 数据），那这个 TCP 连接自然是建立着的。但如果两人只是偶尔通信。那么，其中一个人（或两人同时）想知道对方是否还在，就会定期发送一份邮件（Keep Alive 探测包），这个邮件没有实质内容，只是问对方是否还在，如果对方收到，就会回复说还在（对这个探测包的 ACK 回应）。

需要注意的是，keep alive 技术只是 TCP 技术中的一个可选项。因为不当的配置可能会引起诸如一个正在被使用的 TCP 连接被提前关闭这样的问题，所以默认是关闭的

对于TCP长连接保活是十分必要的，原因如下：
* 系统多在OA网和外网间有防火墙隔离，很多防火墙对一段时间内没有报文活动的socket会自动关闭。

* 对于非正常断开的连接系统并不能侦测到，比如防火墙关闭端口、网线被拔掉、电脑突然奔掉、未关闭应用程序直接关机(服务端无法释放资源)。(调用close(fd)为正常断开，连接对端可以侦测到)


## 二、配置
### 1、linux 配置
在 Linux 操作系统中，TCP Keep Alive 相关的配置可以在 /proc/sys/net/ipv4/ 目录中找到，具体有下面三个（括号中的是默认值）：

* tcp_keepalive_time (7200)
在空闲相应时间（单位是秒）之后，TCP 协议栈将发送 Keep Alive 探测包；

* tcp_keepalive_intvl (75)
开始探测之后每个探测包所间隔的时长，单位同样是秒；

* tcp_keepalive_probes 
探测包的个数。

默认的配置通常不适合一般的 Web 服务器，实际参数会远小于上述默认值。

正如前文所说，TCP Keep Alive 是用来检测 TCP 连接有效性的一种机制，如果一个空闲的 TCP 连接如果失效，那究竟多久能发现？假设采用如下配置：
tcp_keepalive_time = 60
tcp_keepalive_intvl = 10
tcp_keepalive_probes = 6
那将在 60 + 10 * 6 = 120s，即两分钟之后发现一个失效的 TCP 连接。

### 2、Nginx Keep Alive 配置（客户端和nginx,nginx和server ？ todo）

```ini
events {
    use                 epoll;
    worker_connections  102400;
}

http {
    upstream keepAliveService {
        server 10.10.131.149:8080;
        keepalive 20;
    }

    server {
        listen 80;
        server_name keepAliveService;
        location /keep-alive/hello {
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_pass http://keepAliveService;
        }
    }
}
```
如果想要启用 HTTP 长连接，那下面这三项配置都是必须的

* keepalive
顾名思义，这个参数用于控制可连接整个 upstream servers 的 HTTP 长连接的个数，即控制总数。但需要注意的是，这个参数并不控制所有 upstream servers 连接的个数。

* proxy_http_verion
这个用于控制代理后端链接时使用的 HTTP 版本，默认为 1.0。要想使用长连接，必须配置为 1.1。除了 location 以外，这项配置还可以放在 http 和 server 这两级中。

* proxy_set_header
除了要将 HTTP 协议设置为 1.1 以外，还要清空 Connection header 中的值。如果不配置 proxy_set_header Connection ""，则发往 upstream servers 的请求中，Connection header 的值将为 close，导致无法建立长连接。

同 proxy_http_version 一样，这项配置也可以放在 http 和 server 这两级中。

## 三、其他
### 1、实际使用
实际使用上接口平均响应时间方面差距不大，但是在服务器负载方面，应用长连接对降低服务器CPU的负载还可以。

