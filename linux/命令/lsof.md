## Linux 命令神器：lsof

lsof是系统管理/安全的尤伯工具。将这个工具称之为lsof真实名副其实，因为它是指“列出打开文件（lists openfiles）”。而有一点要切记，**在Unix中一切（包括网络套接口）都是文件。**

lsof可以替代netstat和ps的全部工作。**lsof需要root用户的权限来执行(否则显示不全,比如80端口是被root用户使有，普通用户查看不了)。**

```sh
 #来显示与指定端口相关的网络信息
 # sudo losf -i:port
 sudo lsof -i :8081
 
 // 同时显示pid
 sudo lsof -i :80 -P
```

```sh
# 显示所有端口
sudo lsof  -i
# 仅显示TCP连接
sudo lsof  -iTCP
```

```sh
## 找出监听端口
#  sudo lsof  -i -sTCP:LISTEN
```

### 其他使用方法
https://www.jianshu.com/p/a3aa6b01b2e1
