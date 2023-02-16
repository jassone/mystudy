## linter

## 一、官方提供的相关工具

###  1、gofmt-go sdk

go sdk自带的format工具。

问题：在不同版本中表现不一样。

### 2、go vet-go sdk

用于检查Go语言源码中静态错误的简单工具。例如unsafe.Pointer的错误使用，unreachable code等。

分析某个文件

```
go vet main.go
```

分析某个包

```go
go vet floder/*.go
go vet floder/...
```

比如 bodyclose 这个包，可以用来检测 HTTP 响应主体是否关闭，使用方法：

```sh
go vet -vettool=$(which bodyclose) github.com/timakin/go_api/...
```

```sh
//todo
go tool vet -shadow main.go
```

另外，go 还有很多质量工具(可在goland中配置)

* goimports(来检测引用包的问题)

还有一些其他第三方的质量检测工具，帮助我们完善代码：https://github.com/analysis-tools-dev/static-analysis#got

问题：在不同版本中表现不一样。

### 3、golint-已废弃

go团队提供的风格检查工具，用于检查代码风格的错误部分。常见的检查点：comments风格，变量风格，error string等。

```sh
➜  blockchain2 git:(master) ✗ golint ./...
BLC/Block.go:1:1: don't use MixedCaps in package name; BLC should be blc
BLC/Block.go:10:6: exported type Block should have comment or be unexported
```

问题：

- 年久失修，已经废弃。
- 结果并不一定正确，并且无法解决false positives（假阳性）问题。

## 二、社区提供的linters聚合器---golangci-lint

### 1、安装

```sh
go get github.com/golangci/golangci-lint/cmd/golangci-lint
go install github.com/golangci/golangci-lint/cmd/golangci-lint
```

### 2、使用

##### a) 检查当前目录下所有的文件
golangci-lint run 等同于 golangci-lint run ./...

##### b) 可以指定某个目录和文件
golangci-lint run dir1 dir2/... dir3/file1.go`
检查 dir1 和 dir2 目录下的代码及 dir3 目录下的 file1.go 文件

##### c) 可以通过 --enable/-E 开启指定 Linter, 也可以通 --disable/-D 关闭指定 Linter

 ### 2、禁用 lint 检查
有的时候会需要禁用 lint 检查, 可以在需要禁用检测的函数或者语句附近这样使用: //nolint:lll,funlen

在配置里面禁用

```ini
[[issues.exclude-rules]]
path = "(.+)_test.go"
linters = ["typecheck"]
text = "imported but not used"
```

### 1、优势

- 解决了false positives问题，使用//nolint等方式来屏蔽错误提示
- 统一的配置文件，解决了不同工具的配置文件问题
- 整个过程只解析一遍AST，也可以复用文件缓存，大大加快了速度。
- 可插拔linter配置，自由选配linters，自身携带了大部分linters。

### 2、go存在两个编译器前端

