## Golint

Golint 是一个源码检测工具用于检测代码规范，Golint 不同于gofmt, Gofmt用于代码格式化。
Golint会对代码做以下几个方面检查：

- package注释 必须按照 “Package xxx 开头”
- package命名 不能有大写字母、下划线等特殊字符
- struct、interface等注释 必须按照指定格式开头
- struct、interface等命名
- 变量注释、命名
- 函数注释、命名
- 各种语法规范校验等

### 1、Golint安装

```sh
mkdir -p $GOPATH/src/golang.org/x/
cd $GOPATH/src/golang.org/x/
git clone https://github.com/golang/lint.git  
git clone https://github.com/golang/tools.git
```

  到目录$GOPATH/src/golang.org/x/lint/golint中运行

```sh
go install
```

### 2、使用

```sh
golint 项目名称
```

### 3、相关wiki

- https://www.cnblogs.com/gtea/p/16168798.html