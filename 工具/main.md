## 工具汇总

## 一、rpc

### 1、grpcui

```sh
grpcui -plaintext 127.0.0.1:8080 // 打开管理页面


启动go-zero rpc服务后，执行 grpcui -plaintext localhost:9080报如下错误：
Failed to compute set of methods to expose: server does not support the reflection API
解决方法：
go-zero 要开启 dev 或 test 模式， 才会在主方法中注册到反射中 reflection.Register(grpcServer)，否则执行 grpcui 会报错
配置文件下添加 Mode: dev 即可。
```

### 2、grpcurl

```sh
grpcurl -plaintext 127.0.0.1:8080 rpcdemo.Rpcdemo/Ping  // 访问
```

