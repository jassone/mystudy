## nginx限流

## 一、自带的限流模块

Nginx 提供了两种限流手段：一是控制速率，二是控制并发连接数。

### 1、控制速率
我们需要使用 limit_req_zone 用来限制单位时间内的请求数，即速率限制，示例配置如下：

```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
        limit_req zone=mylimit;
    }
}
```
以上配置表示，限制每个 IP 访问的速度为 2r/s，因为 Nginx 的限流统计是基于毫秒的，我们设置的速度是 2r/s，转换一下就是 500ms 内单个 IP 只允许通过 1 个请求，从 501ms 开始才允许通过第 2 个请求。

上面的速率控制虽然很精准但是应用于真实环境未免太苛刻了，真实情况下我们应该控制一个 IP 单位总时间内的总访问次数，而不是像上面那么精确但毫秒，我们可以使用 burst 关键字开启此设置，示例配置如下：

```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server { 
    location / { 
     limit_req zone=mylimit burst=4;
    }
}
```
burst=4 表示缓存区个数，用于缓存还没来得及出来的请求，如果缓冲区也存满了，则拒绝新的请求。

### 2、控制并发数
利用 limit_conn_zone 和 limit_conn 两个指令即可控制并发数，示例配置如下：

```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```
其中 limit_conn perip 10 表示限制单个 IP 同时最多能持有 10 个连接；limit_conn perserver 100 表示 server 同时能处理并发连接的总数为 100 个。

小贴士：只有当 request header 被后端处理后，这个连接才进行计数。

## 二、lua脚本配合限流
![B5145424.png](https://pic.imgdb.cn/item/62af30210947543129cb73b4.png)

每一个Nginx节点都会对应一个独立的Redis节点，由Redis来负责存储于限流相关的配置信息，比如限流名单，错误码，提示信息，限流阀值，限流开关，以及单位时间等。