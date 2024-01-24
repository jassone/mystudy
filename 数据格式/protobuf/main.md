## protobuf

## 一、继承知识

### 1、特点

protobuf之所以小且快，就是因为使用**变长的编码规则**，只保存有用的信息，节省了大量空间。

## 二、细节

### 1、创建

```protobuf
syntax = "proto3";
package user;
option go_package = "./"; // 必须要带上
```

## 相关wiki

- protobuf详解 https://zhuanlan.zhihu.com/p/432875529 

- protobuf简介(数据结构方面)http://t.zoukankan.com/hnxxcxg-p-9405192.html

  

## 工具

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
grpcurl -d @ localhost:19120 task.TaskService/NotifyResult
```

