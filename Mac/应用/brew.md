## brew
各个软件安装目录
/usr/local/Cellar

## 一、命令
```sh
##更新 Homebrew
brew update

##查询要安装的软件是否存在
brew info php

##查看预安装的软件信息和安装后信息
brew info nginx

##安装软件
brew install nginx

## brew tap
brew tap XXXX // 可以为brew的软件的 跟踪,更新,安装添加更多的的tap formulae
// 如果你在核心仓库没有找到你需要的软件,那么你就需要安装第三方的仓库去安装你需要的软件

brew tap // 没有参数会自动更新已经存在的tap并列出当前已经tapped的仓库
// https://segmentfault.com/a/1190000012826983


brew -help 			# 查看帮助命令
brew config			# 查看配置信息
brew list 			# 查看已安装软件包列表
brew cleanup                    # 清理所有包的旧版本
 
brew search [包名] 		# 查询
brew install [包名] 		# 安装
brew info [包名] 		# 查看包信息
brew update [包名] 		# 更新软件包
brew uninstall [包名] 		# 卸载
 
brew services list 		# 列出正在运行的服务
brew services cleanup  		# 清除已卸载应用的无用的配置
 
brew services start [服务名]     # 启动服务
brew services stop [服务名]      # 停止服务
brew services restart [服务名]   # 重启服务

brew commands
```



## 二、资源
* https://brew.sh/index_zh-cn

## 三、相关问题
### 1、brew update 无响应
*  https://www.jianshu.com/p/631e63dab0a0