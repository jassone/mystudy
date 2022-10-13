## go mod(module)

todo****

## 一、相关命令（都需要进入项目中执行）

### 1、基础

```sh
// 初始化项目，生成 go.mod 文件, 
go mod init {项目名称,即module名}  

// 整理现有的依赖
go mod tidy 
1. 引用项目需要的依赖增加到go.mod文件。
2. 去掉go.mod文件中项目不需要的依赖。

// 查看现有的依赖结构
go mod graph 

// 下载 go.mod 文件中指明的所有依赖,在GOPATH/pkg/mod下
go mod download 

// 校验一个模块是否被篡改过,如果将GOPATH/pkg/mod下的相应文件手动改了，这里会验证不通过
go mod verify 

// 编辑 go.mod 文件, 有很9个子命令，可以用go help mod edit查看
go mod edit 
1. go mod edit -fmt  格式化

// 导出项目所有的依赖到vendor目录(当前项目下会生成vendor目录)
go mod vendor 
1. 好处：比如devops中将所有依赖打包，就可以不用到OGPATH中去找，这样就和GOPATH脱钩了，方便打包部署
2. 缺点：会产生冗余依赖包。

go mod why 查看为什么需要依赖某模块
 
```

 ### 2、命令总结

##### a) 列出依赖

- go mod graph
- go mod why
- go list -m all

##### b)添加依赖

- go get
- go build
- go mod edit require
- go mod dowmload

## 二、相关wiki

- go mod使用 | 全网最详细 https://zhuanlan.zhihu.com/p/482014524
- gomod 使用 https://www.jianshu.com/p/d99ca70db58e
- Go Modules详解 https://blog.csdn.net/u011069013/article/details/110114319
