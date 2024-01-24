## gRPC

gRPC由google开发，是一款语言中立、平台中立、开源的远程过程调用系统。

gRPC可以实现微服务，默认使用protocol buffers，这是google开源的一套成熟的结构数据序列化机制（当然也可以使用其他数据格式如JSON）。

## 一、基础知识

### 1、gRPC 四种数据交互模式
* **简单模式（Simple RPC），也叫一元调用**，客户端向服务器发送单个请求并返回单个响应，就像普通的函数调用一样，也就是最常见的客户端请求、服务端响应实现方式；
* **服务端流式 RPC（Server-side streaming RPC）**，客户端向服务器发送请求，并获取多个响应消息。客户端从返回的流中读取，直到没有更多消息。gRPC 保证单个 RPC 调用中的消息排序；

     应用场景：客户端向服务端发送股票代码，服务端把该股实时数据源源不断返回给客户端。
     
* **客户端流式 RPC（Client-side streaming RPC）**，其中客户端写入一系列消息并将它们发送到服务器(多次请求)，再次使用提供的流。一旦客户端完成写入消息，它等待服务器读取它们并返回其响应。gRPC 再次保证单个 RPC 调用中的消息排序；这种模式用的很少，因为实际上基本都是一次性提交过来。

    应用场景：物联网终端向服务器报送数据。
    
* **双向流式 RPC（Bidirectional streaming RPC）**，其中双方使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照他们喜欢的任何顺序进行读写：例如，服务器可以在写入响应之前等待接收所有客户端消息，或者它可以交替读取消息然后写入消息，或其他一些读取和写入的组合。保留每个流中消息的顺序；

### 2、为什么能支持流模式，双向流模式

因为是建立在http2.0基础上的，http2.0天生支持。

## 二、安装protoc
protoc是Protobuf编译器。
### 1、官网下载
- 下载地址：https://github.com/protocolbuffers/protobuf/releases。

- wiki:https://www.cnblogs.com/niuben/p/14212878.html


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

懒人做法：上述可能会出现找不到某些依赖文件，可以直接从一台能访问网络且安装完成的服务器上打包github.com，然后sz、rz到无网络服务器，解压到$GOPATH/src目录下，go build、go install即可。

### 2、安装中遇到的问题

##### a）google.golang.org/protobuf如果比较慢，下载可以找其他资源，比如

地址：https://robinqiwei.coding.net/p/googleprotobuf/git

```sh
git clone https://e.coding.net/robinqiwei/googleprotobuf.git
```

##### b) 如何下载不同版本的proto

```sh
protoc --version
libprotoc 23.4
```

### 四、使用

```sh
protoc --go_out=. --go-grpc_out=. demo.proto // 生成pb文件，进入proto文件目录
```

### 五、相关wiki

* 如何使用grpc https://zhuanlan.zhihu.com/p/161473581
* https://studygolang.com/articles/14336
* grpc安装教程 注意不同版本 https://www.jianshu.com/p/154ea6c13618