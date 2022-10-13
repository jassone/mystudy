## 下载和安装
官网正常安装即可

## 一、设置环境变量
```sh
// 增加
export GOPATH=$HOME/webdata/go  （workspace，工作空间）
export GOBIN=$GOPATH/bin
export PATH=$PATH:/usr/local/go/bin:$GOBIN

//GOROOT    默认  /usr/local/go，可不设置

source /etc/profile ##刷新环境变量

//或者
vi ~/.bash_profile  添加

//运行
source  ~/.bash_profile     刷新环境变量

//环境变量
vi /etc/profile  // 所有用户的
```

## 二、其他配置

### 1、设置module

```sh
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct // 使用七牛云的
```

注意：go env -w会将配置写到 `GOENV="/Users/bian/Library/Application Support/go/env"`