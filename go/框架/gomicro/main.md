## main



- 注意其中的--namespace参数,我们每一个微服务都属于一个命名空间,通过api暴露出来该命名空间后,满足`go.micro.srv.*`格式的微服务都可以访问。

  auth.Namespace(string)：

  微服务命名空间，以原数据Micro-Namespace存储在context中。

  设置namespace命名空间可将各个环境隔离开



protoc --proto_path=$GOPATH/src:. --micro_out=. --go_out=. proto/helloworld.proto

http://127.0.0.1:8080/helloworld/Call?name=ddd



## 相关文档

- 教程  https://learnku.com/docs/go-micro/3.x
- 博客 https://laravelacademy.org/
- 官网 https://github.com/go-micro/go-micro
- 官网demo https://github.com/go-micro/examples





http://e.betheme.net/article/show-140787.html?action=onClick

https://zhuanlan.zhihu.com/p/437396161?utm_id=0

