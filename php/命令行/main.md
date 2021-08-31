##  php
* php -i 查看php信息

* php -m 查看php模块信息
* php --ri XXX  查看扩展的版本信息
```
php --ri swoole
``` 
* php -f 解析并运行给定的文件名，如果后面是文件名即可省略

```
php [-f] index.php
```

* php -r 解析PHP代码


```
php -r "echo 'hello world';"
php -r 'echo 123;'
```


##### 其他
* -a 交互运行php
* -c <path>|<file> 运行时，指定php配置文件(XXX.ini)

```
php -c ./custom-php.ini my_script.php
```
* -n 不使用配置文件
* -d foo[=bar] 设置php.ine配置文件中foo的值为bar
* -e 为调试器等生成扩展信息
* -f <file> 解析运行脚本文件
* -h 帮助
* -i php信息(phpinfo())
* -l 检测代码语法
* -m 显示已加载的模块
* -r \<code> 运行php代码
* -B \<begin_code> 运行php代码在输入行之前
* -R \<code> 每行输入行运行一次php代码
* -F <file> 每行输入行解析运行脚本文件
* -E <end_code> 处理完所有输入行，运行php代码。
* -H 隐藏拓展工具传入的参数
* -S <addr>:<port> 使用内置的web服务运行
* -t <docroot> 为内容之web服务指定项目根目录（document root）
* -s 高亮输出
* -v php版本号
* -w 输出源和剥离的注释和空白
* -z <file> 加载zend拓展文件
* args... 传递参数到脚本. 使用 -- args
* --ini 显示配置文件名称
* --rf <name> 显示某函数的信息
* --rc <name> 显示某类的信息
* --re <name> 显示某拓展的信息
* --rz <name> 显示某zend拓展的信息
* --ri <name> 显示某拓展的配置信息
