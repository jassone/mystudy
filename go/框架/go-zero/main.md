## main

## 一、安装

```sh
go get -u github.com/zeromicro/go-zero
go install github.com/zeromicro/go-zero/tools/goctl@latest
```

## 二、快速运行

### 1、api服务

```sh
goctl api new greet
cd greet
go mod init  // ??
go mod tidy
go run greet.go -f etc/greet-api.yaml

curl -i http://localhost:8888/from/you
```

或者

```go
func main() {
    var restConf rest.RestConf
    conf.MustLoad("etc/helloworld.yaml", &restConf)
    s, err := rest.NewServer(restConf)
    if err != nil {
        log.Fatal(err)
        return
    }

    s.AddRoute(rest.Route{ // 添加路由
        Method: http.MethodGet,
        Path:   "/hello/world",
        Handler: func(writer http.ResponseWriter, request *http.Request) { // 处理函数
            httpx.OkJson(writer, "Hello World!")
        },
    })
    
    defer s.Stop()
    s.Start() // 启动服务
}
```



### 2、rpc服务

https://go-zero.dev/docs/tasks/cli/grpc-demo

或者

```sh
# 创建 demo 服务目录
$ mkdir demo && cd demo

# go mod 初始化
$ go mod init demo

# 生成 greet.proto 文件
$ goctl rpc -o greet.proto

# 生 pb.go 文件
$ protoc greet.proto --go_out=. --go-grpc_out=.

# 创建 server 目录
$ mkdir server && cd server

# 新增配置文件
$ mkdir etc && cd etc
$ touch greet-server.yaml

# 新增 server.go 文件
$ touch server.go
```



```go
func main() {
  var clientConf zrpc.RpcClientConf

	// 这里不能用相对路径，很奇怪？？？？ 因为需要在上几层目录中执行才行
	conf.MustLoad("/Users/bian/workdata/go/go-zero/rpcdemo/etc/rpcdemo.yaml", &clientConf)
	conn := zrpc.MustNewClient(clientConf)

	client := rpcdemoclient.NewRpcdemo(conn)
	resp, err := client.Ping(context.Background(), &rpcdemoclient.Request{})
	if err != nil {
		log.Fatal(err)
		return
	}

	log.Println(resp)
}
```



mongo 带缓存的代码???

Sqlc 的带缓冲的，里面包含了sqlx，既执行完sqlc再执行sqlx



## 二、其他

- 在主api文件中引入其他api，用这一个api去生成即可

## 三、功能点

- 客户端请求时，支持拦截器



## 相关文档

- 还比较全 https://www.w3cschool.cn/gozero/
- github 文档 https://github.com/zeromicro/go-zero/blob/master/readme-cn.md
- 官方文档： https://go-zero.dev/
- 工程实践： https://github.com/Mikaelemmmm/go-zero-looklook/blob/main/README-cn.md
