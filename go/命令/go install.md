## go install

go install 用于编译并安装它指定的代码包以及代码包依赖的其他包。作用有两个：

- 如果是库文件(无main包)，编译后的库文件，会生成目标库文件(编译好的 .a 文件（归档文件或静态链接库文件）)，放置到GOPATH/pgk目录下。
- 如果是可执行文件(有main包)，编译后的可执文件，会生成目标可执行文件，并且放置到GOPATH/bin目录下。

![323234325.png](https://pic.imgdb.cn/item/630f742216f2c2beb1e22f60.png)

### 1、和go build的区别

其实 go build 的绝大多数命令都可以用于 go install 命令

- **go install比 go build 多做了一件事，就是生成目标库文件(.a文件)**。 
- go build生成的可执行文件在当前目录下，而go install生成的可执行文件放在GOPATH/bin目录下。

### 2、扩展：什么是归档文件

归档文件在Linux下就是扩展名是.a的文件，也就是archive文件，是程序编译后生成的静态库文件。
