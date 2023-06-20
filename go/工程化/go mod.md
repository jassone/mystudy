## go mod(module)

## 一、go mod如何工作

### 1、版本的两种表达方式

##### a) 语义化版本（semver）

```
golang.org/x/sys v0.1.0 // indirect
```

v${MAJOR}.${MINOR}.${PATCH}

**MAJOR版本不同则认为是两个不同仓库**。根据语义版本语义，主要版本与次要版本不同。主要版本可能会破坏向后兼容性。从Go模块的角度来看，主要版本是 完全不同的软件包。两个不兼容的库版本是两个不同的库。

##### b) 基于某一commit的伪版本号

```
golang.org/x/lint v0.0.0-20210508222113-6edffad5e616 // indirect
```

v${基本版本前缀}-${commit UTC时间}-${commit hash前12位}

### 2、拉取package

go module 安装 package 的原則是先拉最新的 release tag，若无tag则拉最新的commit

- 如果当前commit之前有打过tag，则patch+1，
- 如果没有加tag，即没有版本的控制。默认是v0.0.0后面接上时间和commitid。如下：

```text
gomodone@v0.0.0-20200517004046-ee882713fd1e
```

官方不建议这样做，没有进行版本控制管理。使用tag，进行版本控制

```text
git tag v1.0.0
git push --tags
```

操作完，我们的module就发布了一个v1.0.0的版本了。

### 3、管理go.mod的两个重要工具

- go mod tidy
- go get 

### 4、go mod版本选择算法

使用Minimal version selection (MVS)算法：**对每个依赖，选择其所有被依赖版本中最高的那个版本**。

![20221117134852.jpg](https://pic.imgdb.cn/item/6375cbfc16f2c2beb1da2fc0.jpg)

这里会C会用1.4，D会用1.2（D虽然有更高的1.3,但是不是依赖链路中的）

## 二、相关命令（都需要进入项目中执行）

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

// 导出项目所有的依赖到vendor目录(当前项目下会生成vendor目录)，默认是忽略vendor的。
go mod vendor
1. 好处：比如devops中将所有依赖打包，就可以不用到OGPATH中去找，这样就和GOPATH脱钩了，方便打包部署
2. 缺点：会产生冗余依赖包。

如果构建程序的时候，希望使用vendor中的依赖，
$ go build -mod vendor

go mod why 查看为什么需要依赖某模块
 
```

 ### 2、命令总结

##### a) 列出依赖

- go mod graph

- go mod why

- go list -m all （只列出所有依赖）

  go list -m -u all 来检查可以升级的package

##### b)添加依赖

- go get

  | @upgrade | 默认行为(@upgrade可不加)<br/>若当前是semver：更新到最新的minor semver（所以可能拉不到最新版本，因为不会切换为位伪版本）<br/>若当前是伪版本：更新到最新minor semver(若有)或当前base version的最新commit |
  | -------- | ------------------------------------------------------------ |
  | @lastest | 修改依赖至最新的minor semver,可能会降低版本(比如v0.1.1->v0.1.0。因为是使用最新的语义化版本，而不是伪版本，所以可能拉不到最新的版本) |
  | @patch   | 更新到最新的patch版本                                        |
  | @none    | 删除依赖                                                     |
  | @v1.2.3  | 更新到sermver。升级到指定的版本号version(比如go get github.com/jacksonyoudi/gomodone@v1.0.1) |
  | @2ddgege | 拉取特定的commit，至少7位                                    |
  | @master  | 分支的最近一次commit                                         |

  - 若major版本>1,go get 需要加版本后缀：go get example.com/pkg/v2
  - **go get -u(=patch)  : 升级所有依赖，会同时更新该依赖中所有参与编译的依赖到最新版本。所以如无必要，不要使用-u。**
  -  go get -u(=patch)  将会升级到最新的次要版本或者修订版本号(x.y.z, z是修订版本号， y是次要版本号,即它将从1.0.0更新到例如1.0.1，或者，如果可用，则更新为1.1.0）
  - go < 1.18时，若go get 的目标package为main,还会同时编译安装二进制到GOPATH/bin下。而高版本则用go install代替，go get仅用于依赖管理。

  

- go build

- go mod edit require

- go mod dowmload

### 3、go.mod文件

##### a) **使用replace替换无法直接获取的package**

比如在 go.mod 文件中使用 replace 指令替换成github上对应的库。

```text
replace golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
```

##### b) 注释说明

```
require (
   github.com/jassone/note v0.0.0-20191229100458-0e65abf2fc8f // indirect
   golang.org/x/tools v0.2.0 // indirect
   example.com/m v4.1.2+incompatible
)
```

// indirect标记 ：非本项目所直接依赖，但在本项目中指定该依赖项的版本。可能的原因：

- 依赖项没有使用go mod
- 不是所有依赖项都在go.mod中
- 手动为依赖的依赖指定较新的版本

+incompatible标记：该依赖未使用go mod管理依赖，产生的影响：

- 不影响使用，但该依赖的依赖的版本没有被显示指定
- **不再认为不同major版本号为不同项目，会优先使用最新的**

### 4、常用工具和方法

##### a) 查询所有依赖项之间的依赖关系 & 获取参与编译的依赖项代码

- go mod graph
- go mod vendor

##### b）临时修改依赖库中的代码进行调试测试

- clone仓库到本地 & checkout对应版本 & replace
- go.mod文件中 replace github.com/foo/bar => ../../github.com/foo/bar

##### c）批量更新依赖

- go get github.com/..

##### d）仅安装/运行二进制但不更新当前项目的依赖

- go install example.com/cmd@lastest(go >=1.16)
- go run example.com/cmd@lastest(go >=1.17)
- go 1.18后go get将只用于更新依赖而不安装二进制

## 三、配置

### 1、设置go module

```sh
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct # 使用七牛云的
go env -u GOPROXY #取消代理
```

注意：go env -w会将配置写到 `GOENV="/Users/bian/Library/Application Support/go/env"`

- GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。

- GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。

- GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。启用module时分为两种情形：

  a 当前目录在GOPATH/src之外且该目录包含go.mod文件
  b 当前文件在包含go.mod文件的目录下面。

## 四、相关问题

### 1、项目庞大复杂，go.mod臃肿，如何解决

- 通常进行拆库，特殊场景下可能会对子目录采用submodule

### 2、作为维护者，如何优雅地告知使用者某个版本存在bug？

- retract标记(go >= 1.16)

直接删tag并不友好。

### 3、如何组织数量庞大的module

这里依然推荐使用GOPATH按照路径管理所有依赖的方式

- 各工程的组织结构清晰
- 可以从工程的相对路径获取go mod

### 4、为什么有一些工程不使用go mod

```
比如
github.com/dgrijalva/jwt-go v3.2.0+incompatible
```

- 在go mod推广前，major版本已>1，如果使用go mod ，则无法通过go get升级版本，需要加上v3版本后缀才行(开发者会经常忘记)。
- 便于go get 直接拉到v3版本

经验：在复杂的业务场景中，新项目尽量不要使用>1的major版本号。如果真的需要可以新开个项目，或者在新路径下维护一个新版本。或者是使用域名重定向，使用携带版本号的域名。

### 5、几个案例

![20221117165656.jpg](https://pic.imgdb.cn/item/6375fa9416f2c2beb12f9804.jpg)

经验：

- 特殊情况下可以手动指定indirecty依赖(比如上面A的go.mod中配置上X的v2版本)。
- 无论依赖库是否使用go mod，我们的项目中都应该使用go md。

![20221117172420.jpg](https://pic.imgdb.cn/item/6375fe7916f2c2beb1370177.jpg)

### 6、 循环依赖

公共库直接应明确分工，避免大杂烩，避免循环依赖

### 7、如何把一个库从v1.1升级到v2.0

在新的commit中在根目录新建v2目录，将v1文件都复制到v2中，go.mod中path修改为v2，打上v2.0tag

wiki：https://go.dev/blog/v2-go-modules

## 五、相关wiki

- go mod使用 | 全网最详细 https://zhuanlan.zhihu.com/p/482014524 （推荐）
- gomod 使用 https://www.jianshu.com/p/d99ca70db58e
- Go Modules详解 https://blog.csdn.net/u011069013/article/details/110114319
