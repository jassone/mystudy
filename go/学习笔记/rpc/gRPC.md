## gRPC
## 一、基础知识
### 1、gRPC 四种数据交互模式
* 简单模式（Simple RPC），客户端发起请求并等待服务端响应；
* 服务端流式 RPC（Server-side streaming RPC），客户端发起一个请求到服务端，服务端返回一段连续的数据流响应；
* 客户端流式 RPC（Client-side streaming RPC），与服务端流式相反，客户端流式是客户端不断地向服务端发送数据流，最后由服务端返回一个响应；
* 双向流式 RPC（Bidirectional streaming RPC），客户端和服务端可同时向对方发送数据流，同时也可以接收数据；

## 二、安装protoc
protoc是Protobuf编译器。
### 1、官网下载
下载地址：https://github.com/protocolbuffers/protobuf/releases。

wiki:https://www.cnblogs.com/niuben/p/14212878.html
 
### 2、brew安装
```sh
brew install protobuf
```
生成命令  /usr/local/bin/protoc

## 三、gRPC使用
使用gRPC分为三步
* 编写.proto文件
* 利用工具将.proto文件生成对应语言的代码
* 根据生成的代码编写服务端和客户端的代码

### 1、安装
* go get github.com/golang/protobuf/proto
* go get google.golang.org/grpc（无法使用，用如下命令代替）
    git clone https://github.com/grpc/grpc-go.git \$GOPATH/src/google.golang.org/grpc
    git clone https://github.com/golang/net.git \$GOPATH/src/golang.org/x/net
    git clone https://github.com/golang/text.git \$GOPATH/src/golang.org/x/text
    go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
    git clone https://github.com/google/go-genproto.git \$GOPATH/src/google.golang.org/genproto
* cd \$GOPATH/src/
* go install google.golang.org/grpc
* go get github.com/golang/protobuf/protoc-gen-go

### 2、protoc-gen-go安装
protoc-gen-go是go版本的 Protobuf 编译器插件，

go get -u github.com/golang/protobuf/protoc-gen-go 便可以在$GOPATH/bin目录下发现这个工具。

当Linux系统无法访问网络时：首先在github.com/golang/protobuf上下载protoc-gen-go和proto，（最好将其放在$GOPATH/src目录下）然后进入protoc-gen-go目录，执行go build、go install即可在$GOPATH/bin目录下发现这个工具。

前提是必须首先将$GOPATH/bin路径添加到环境变量$PATH中。

懒人做法：上述可能会出现找不到某些依赖文件，直接从一台能访问网络且安装完成的服务器上打包github.com，然后sz、rz到无网络服务器，解压到$GOPATH/src目录下，go build、go install即可。

### 2、安装中遇到的问题
##### a）google.golang.org/protobuf如果比较慢，下载可以找其他资源，比如
地址：https://robinqiwei.coding.net/p/googleprotobuf/git

```sh
git clone https://e.coding.net/robinqiwei/googleprotobuf.git
```

### 四、相关wiki
* https://zhuanlan.zhihu.com/p/161473581
* https://studygolang.com/articles/14336