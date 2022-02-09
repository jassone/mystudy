## 启动关闭等
```sh
#启动  
redis-server  /etc/redis.conf 
#（后面配置一定要加，因为不知道读取了默认的哪个配置）

#连接(默认链接本地，6379端口)
redis-cli  

#连接(指定ip和端口)
redis-cli -h 127.0.0.1 -p 6379
```

## 连接
### 1、TCP连接
客户端和服务器通过 **TCP** 连接来进行数据交互， 服务器默认的端口号为 6379 。

### 2、unix socket套接字连接
在redis.conf文件中取消注释两条语句:

```ini
unixsocket /tmp/redis.sock 
unixsocketperm 700
```

然后运行redis,创建redis连接时用redis.new (:path =>"/tmp/redis.sock")，即可
 
