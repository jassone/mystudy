## go工程化变迁

Golang 依赖管理机制发展到现在，主要经历了 GOPATH、govendor、go mod三个时期。而 go mod 发展到现在，已经能够较好地解决依赖管理中的问题了。

## 一、GOPATH

### 1、特点

- 所有依赖库按路径组织再src下，
- 编译时直接使用src下的代码
- go get会下载代码到src下

### 2、带来的问题

- 不同环境下依赖库版本不一致
- 无法很好地控制依赖库的版本

## 二、govendor

### 1、特点

- 所有依赖库按照路径组织再项目的vendor目录
- 编译使用vendor中的代码
- 通过govendor工具来更新vendor下的代码

### 2、带来的问题

- 解决了GOPATH方式的第一个问题

## 三、go mod

### 1、特点

- 所有依赖代码按路径和版本号组织在GOPATH/pkg/mod目录下
- 采用go.mod文件描述依赖项的版本
- 通过go get/mod等命令管理依赖

### 2、带来的问题

- 解决了GOPATH项目的两个问题





go mod 的核心原理主要包含三部分内容：

- 版本的两种表达方式：semver (Semantic Versioning) 语义化版本、基于某一 commit 的伪版本号；
- 管理 go.mod 的两个重要工具：go get & go mod；
- 版本选择算法：Minimal Version Selection(MVS)。

在实际工程实践中，开发者们在 go.mod 文件中还会经常遇到一些非理想状态：

- **// indirect** 标记：非本项目所直接依赖，但在本项目中指定该依赖项的版本；
- **+incompatible** 标记：该依赖未使用 go mod 管理依赖。

大家需要注意这些依赖项会存在一些特殊的行为。此外，我们还需要掌握一些常用工具和方法。

在依赖分析和处理层面主要有：

- 查询依赖关系和依赖库代码：go mod graph & go mod vendor；
- 修改依赖库：replace；
- 批量更新和安装/运行二进制：go get ...后缀、携带版本后缀的 go run/install。

在工程管理实践经验层面主要有：

- 庞大项目的依赖管理：拆库或 submodule；
- 版本撤回标记：retract；
- 本地工程目录组织：在 $GOPATH 按路径管理。