## Redis整理
## 一、启动关闭
### 1、启动关闭
```sh
#启动  
redis-server  /etc/redis.conf 
#（后面配置一定要加，因为不知道读取了默认的哪个配置）

#连接(默认链接本地，6379端口)
redis-cli  

#连接(指定ip和端口)
redis-cli -h 127.0.0.1 -p 6379
```

### 2、如何不重启redis修改配置
例如
redis-cli> CONFIG SET appendonly yes
但是不要忘记修改配置文件，否则真的重启会丢失上面配置

## 二、连接
### 1、TCP连接
客户端和服务器通过 **TCP** 连接来进行数据交互， 服务器默认的端口号为 6379 。

### 2、unix socket套接字连接
在redis.conf文件中取消注释两条语句:

```ini
unixsocket /tmp/redis.sock 
unixsocketperm 700
```

然后运行redis,创建redis连接时用redis.new (:path =>"/tmp/redis.sock")，即可

## 五、其他
todo
redis大Key一般达到多少，性能就下降了？

1. 因为redis用的动态字符串(string类型)的库在每次分配空间的时候会增加一倍的可用空闲空间，存储大的字符时占空间
2. **redis各个命令在不同版本使用情况不同，请务必要知晓当前使用的redis版本，按照手册使用.**
