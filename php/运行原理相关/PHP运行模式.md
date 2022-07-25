

## PHP运行的5种模式

查看当前运行模式

```php
echo php_sapi_name();
```

比如：fpm-fcgi(fastcig+fpm)，cli，cgi-fcgi(cgi)。

##  一、CLI模式(就是命令行模式)

### 1、和其他web模式不一样的地方有：

- 没有超时时间
- 默认关闭buffer缓冲
- STDIN|STDOUT|STDERR 标准输出/输出/错误 的使用
- echo var_dump phpinfo 等直接输出到控制台
- 可使用的类 函数不同
- php.ini配置不同

##  二、CGI模式(Apache) - (CGI/FastCGI)

*apache*调用外部执行器*php.exe*执行*php*代码，并将解释后生成的*html*代码和原*html*整合，再传递给*apache*服务器。

### 1、示例配置

```ini
# PHP5版写法
ScriptAlias /php/ "D:/php/"
AddType application/x-httpd-php .php
Action application/x-httpd-php "/php/php-cgi(.exe)"
```

### 2、优点：

因为是外部加载执行器，php代码执行出错不会导致*apache*崩溃。

### 3、缺点

其在执行时每次都需要重新解析*php.ini*、重新载入全部*dll*扩展并重初始化全部数据结构，运行速度非常慢。

## 三、FastCGI模式（几乎通用Apache/nginx/lighttp）

fast-cgi 是cgi的升级版本，FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一 次。PHP使用PHP-FPM(FastCGI Process Manager)，全称PHP FastCGI进程管理器进行管理。

![120212_eaA7_1384334.png](https://pic.imgdb.cn/item/611938555132923bf8124a5a.png)

## 四、ISAPI运行模式(IIS独有，window下的Apache)



## 五、模块加载运行模式(LoadModule， Apache独有)

又可细分为两种。

### 1、 mod_php (apache默认运行php方式，apache 2.0 handler)

**该方式其实是最快的的**。在Apache的配置文件(httpd.conf)中添加如下配置(具体配置见apache章节)：

```ini
LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
```
其实原理都是，用LoadModule来加载php7_module，**就是把php作为apache的一个内置的子模块来运行(自带php解释器**)。让apache http服务器本身能够支持php语言，不需要每一个请求就启动php解释器来解释php。

那么，php7_module是如何将数据传给php的解析器来解析php代码的呢？
答案是：sapi

用一张图来看apache、php、sapi三者之间的关系(**还是在apache的模块中处理**)： 

![640.webp](https://pic.imgdb.cn/item/6119136a5132923bf82fb5d2.png)


从上面图中，**我们看出了sapi就是这样的一个中间过程，sapi提供了一个和外部通信的接口，使得PHP可以和其他应用进行交互数据（apache，nginx等）。**

**php默认提供了很多种SAPI，常见的提供给apache和nginx的php7_module、CGI、FastCGI，给IIS的ISAPI，以及Shell的CLI。**

**httpd是Apache超文本传输协议(HTTP)服务器的主程序。被设计为一个独立运行的后台进程，它会建立一个处理请求的子进程或线程池。**

所以，以上的apache调用php执行的过程如下(**还是在apache的模块中处理**)：

`apache -> httpd -> php7_module -> SAPI -> php`

这种模式将php模块安装到apache中，每一次apache请求，都会产生一条进程，这个进程就完整的包括php的各种运算计算等操作。

缺点：

- apache每接收一个请求，都会产生一个进程来连接php通过sapi来完成请求，如果一旦用户过多，并发数过多，服务器就会承受不住了。
- 把php当做一个模块加载到apache中，出问题时很难定位是php的问题还是apache的问题。而且当php模块出现问题可能会导致Apache一同挂掉
- 每个Apache进程都需要加载mod_php，而不论这个请求是处理静态内容还是动态内容，这样导致浪费内存，效率下降。
-  **一旦修改了php.ini,需要重启apache，这使得无法进行平滑配置变更。**

### 2、mod_fcgid模块 (Apache/2.2.15(Win32) mod_fcgid/2.3.6)

- mod_fastcgi及mod_fcgid，则是把php解析与apache2分离了。mod_fcgid是mod_fastcgi的升级，推荐用 mod_fcgid。

- mod_fcgid就是在apache下支持fastcgi协议的模块。mod_cgi即cgi的实现，现在很少用了。
- mod_fcgid可以对对FastCGI进程进行管理。

原来的mod_fastcgi因为实现方式的限制，所以可能会创建了很多不必要的进程，而实际上只需要更少的进程就能处理同样的请求。mod_fastcgi的另外一个问题是每一个CGI的多个进程都共享同一个管道文件，所有到同一个fastcgi的通讯都通过这个同名的管道文件进行， 这样当出现通讯错误的时候，根本不知道正在通讯的是哪一个fastcgi，于是也没有办法将这个有问题的进程杀死。

mod_fcgid尝 试使用共享内存来解决这个问题。共享内存里面有当前每个fastcgi进程的信息（包括进程号，进程使用的管道文件名等），当 每次尝试请求fastcgi工作的时候，Apache将会首先在共享内存里面查询，只有在共享内存里面发现确实没有足够的fastcgi进程了，才会创建 新的进程，这样可以保证当前创建的进程数量刚好能够处理客户的请求。另外，由于每一个fastcgi进程使用不同名称的管道文件，所以可以在通讯失败的时 候知道到底哪个fastcgi进程有问题，而能够尽早的将其剔除。

因为PHP现在还不能保证所有的扩展代码都是线程安全的，所以并不建议在Apache2的线程模式 下使用mod_php。而以FastCGI方式运行的PHP则是其中一个解决办法。另外，使用mod_fcgi还可以在不修改任何PHP代码的情况下，获 得数据库连接池的功能，大大减少PHP进程到数据库的连接。

### 3、总结：

- mod_php是apache的内置php解释模块，使用prefork方式，不需要额外的进程来做通讯和应用解释，显然mod_php比mod_fcgid这样方式性能要好得多。


## 六、目前状况

随着技术的不断升级，单纯的Apache加php模块的方式已不再主流，而是替换为Apache加mod_fcgid，以及后来的php_fastcgi和nginx加php-fpm的方法。

总结一下这个升级的过程： 
![643.webp](https://pic.imgdb.cn/item/61191c285132923bf85b618b.png)

<font color="red">如果要搭建一个高性能的PHP WEB服务器，目前最佳的方式是Apache/Nginx + FastCGI + PHP-FPM(+PHP-CGI)方式了。 </font>
