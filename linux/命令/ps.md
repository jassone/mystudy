## ps
Linux中的ps命令是Process Status的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，

就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

### 查看服务情况
```sh
ps -ef |grep httpd 
```