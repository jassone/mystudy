## CLI SAPI 和 CGI SAPI 
## 概述

SAPI（Server abstraction API 服务抽象接口）提供了一个接口，使得PHP可以和其他应用进行交互数据，具体点说是提供了一个和外部通信的接口。

常见的：给apache的mod_php7，CGI，给IIS的ISAPI，还有Shell的CLI。

CLI SAPI（Server Application Programming Interface，服务端应用编程端口），名为 CLI，意为 Command Line Interface，即命令行接口。

> 即常用的命令’php‘ 

CGI(Common Gateway Interface) SAPI 公共网关接口

> 即命令’php-cgi‘

### 1、相同点

Linux下安装好php，会有“php”和“php-cgi”这两个可执行程序(在win下是php.exe和php-cgi.exe)，这两个程序其实基本上是一样的，都是“php解释器”(就是php的核心)，就是能把你写的php代码进行解释最后输出代码的结果。

### 2、不同点

* php属于cli接口(client客户端接口)，只能用命令去调用，而php-cgi则提供了fastCGI接口，**fastCGI接口是一种“网络接口”，你可以通过网络的方式去调用它。**

    比如nginx调用php-cgi可以用“fastcgi_pass 127.0.0.1:9000;”这样调用，或者换成其他机子地址，这样去调用其他服务器的php-fpm。

* 与 CGI SAPI 不同，CLI SAPI输出没有任何头信息

* 使用 getcwd() 输出当前脚本运行的目录， CGI SAPI是以文件所在目录为基准输出，而 CLI SAPI 则是以当前运行这个命令的目录为基准输出。

* 出错时CLI输出纯文本的错误信息（非 HTML 格式）

* CLI SAPI可以独立于Web Server运行

* CLI SAPI 强制覆盖了 php.ini 中的某些设置，因为这些设置在命令行没有意义，下面设置在CLI强制覆盖
    1 html_errors: false，取消HTML错误信息
    2 implicit_flush: true，print和echo的输出会立即写到输出端，而不做任何缓冲，可使用output buffering实现输出缓冲
    3 max_execution_time: 0，命令行运行的php脚本可能等待输入等，运行时间可能很长
    4 register_argc_argv: true，总是可以在命令行模式下访问命令行参数
## 二、CLI SAPI 运行 PHP代码

##### 指定文件

两种方式：

```sh
php my_script.php
php -f my_script.php
```
可以选择任何文件来运行，指定的 PHP 脚本并非必须要以 .php 为扩展名，它们可以有任意的文件名和扩展名。

##### 命令行直接运行PHP代码

尽量使用单引号，因为双引号中的$符号会被认为是变量而替换
```sh
php -r "echo 'hello world';"
php -r 'echo 123;'

// 报错
php -r "var_dump($argv);" app 
```

## 三、CGI SAPI运行PHP代码

* php-cgi -f 解析并运行给定的文件名,不能直接执行PHP语句 
* php-cgi -i  和代码中打印的phpinfo()一样 

## 四、linux 下CLI安装位置

在默认情况下，当运行 make 时，CGI 和 CLI 都会被编译并且分别放置在 PHP 源文件目录的 sapi/cgi/php 和 sapi/cli/php 下。可以注意到两个文件都被命名为了 php。在 make install 的过程中会发生什么取决于配置行。如果在配置的时候选择了一个 SAPI 模块，如 apxs，或者使用了 --disable-cgi 参数，则在 make install 的过程中，CLI 将被拷贝到 {PREFIX}/bin/php，除非 CGI 已经被放置在了那个位置。

## 五、相关wiki

- PHP内核之SAPI:Apache2 SAPI分析 https://blog.csdn.net/hao508506/article/details/52344698 