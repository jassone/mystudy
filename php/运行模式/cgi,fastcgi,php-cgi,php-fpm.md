* ## 名词解释
### CGI
CGI是为了保证web server传递过来的数据是标准格式的，方便CGI程序的编写者。

## 两个命令的区别---php和php-cgi
Linux下安装好php，会有“php”和“php-cgi”这两个可执行程序(在win下是php.exe和php-cgi.exe)，这两个程序其实基本上是一样的，都是“php解释器”(就是php的核心)，就是能把你写的php代码进行解释最后输出代码的结果。

### 不同点就在于提供的接口不同

* php属于cli接口(client客户端接口)，只能用命令去调用，而php-cgi则提供了fastCGI接口，fastCGI接口是一种“网络接口”，你可以通过网络的方式去调用它。

    比如nginx调用php-cgi可以用“fastcgi_pass   127.0.0.1:9000;”这样调用，其中的“127.0.0.1”你完全可以换成非本机ip，比如你在B服务器(ip为23.45.67.78)的9000上运行了一个php-fpm，那么你A服务器的nginx可以用“fastcgi_pass   23.45.67.78:9000;”这样去调用B服务器的php-fpm。

## php-cgi与php-fpm的相同和区别

##### 相同
* php-fpm跟php和php-cgi都能解释php代码，
* php-cgi和php-fpm可以通过“网络”来调用，而所使用的网络协议叫“fastCGI协议”

##### 不同
* php-fpm就是php-cgi的升级版（并非简单的在php-cgi的基础上升级，而且通过直接采用第三方代码的方式，实质上是用php-fpm“替换”了php-cgi，而不是简单的升级，但我们可以理解为升级),php-fpm比php-cgi高级很多
* php-cgi不是FastCGI进程管理器,php-fpm才是
* php-fpm是一个独立的SAPI,其管理的不是php-cgi,也就是说php-fpm跟php-cgi无关,php-fpm内置php解释器,php-fpm的子进程是自己fork出来的,并不会调用php-cgi,你把系统中的php-cgi删了也不会影响到php-fpm服务的正常运行 


简单来说，就是php-cgi有很多缺点，有些大牛觉得完全可以改进它，于是就有人写出了php-fpm，最初php-fpm是需要调用php-cgi来解释php代码的，php-fpm只起到进程管理的作用，但是因为php-fpm这个民间第三方写的工具实在比php-cgi好太多了，php官方在php5.4时就把它集成到了php官方发布的包中，并且php-fpm不需要再依赖php-cgi，直接把php解释器的功能集成进php-fpm了。

需要强调一下：win不支持php-fpm，因为php-fpm是使用Linux的fork()来做的，所以win下面基本上还是使用php-cgi.

## 补充
要对FastCGI进程进行管理,需要使用特别的模块
* Apache的mod_fcgid(题外话：php还可以作为apache的子模块运行)
* IIS的PHP Manager
* Nginx只负责反向代理/请求转发,不负责管理php-cgi进程,所以Nginx一般配合能够自行管理工作进程(子进程)的php-fpm使用.
