## netstat
netstat 是一个告诉我们系统中所有 tcp/udp/unix socket 连接状态的命令行工具。它会列出所有已经连接或者等待连接状态的连接。 该工具在识别某个应用监听哪个端口时特别有用，我们也能用它来判断某个应用是否正常的在监听某个端口。

### 查看某个端口号
* netstat -tunlp | grep 8000

如果是mac
* 查询inet：netstat -anvf inet | grep 80
* 查询TCP：netstat -anvp tcp | grep 80
* 查询UDP：netstat -anvp udp | grep 80
