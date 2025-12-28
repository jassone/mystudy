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
vi ~/.bash_profile  添加   // 当前在这下面

//运行
source  ~/.bash_profile     刷新环境变量

//  macOS Catalina(10.15) 开始，默认 shell 从 bash 改为了 zsh
只会加载 ~/.zshrc ，可以子啊该文件后添加：
if [ -f ~/.bash_profile ]; then
    source ~/.bash_profile
fi


//环境变量
vi /etc/profile  // 所有用户的

// 设置环境变量
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
```

## 二、其他配置

### 1、设置module



