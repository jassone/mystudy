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