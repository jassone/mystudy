

## PHP运行的5种模式
###  1 CLI模式
###  2 CGI模式
###  3 FastCGI模式
###  4 ISAPI运行模式
###  5 apache模块运行模式(细分两种)
##### 第一种： mod_php
在Apache的配置文件(httpd.conf)中添加如下配置：
```
LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so
``` 
其实原理都是，用LoadModule来加载php7_module，就是把php作为apache的一个内置的子模块来运行。让apache http服务器本身能够支持php语言，不需要每一个请求就启动php解释器来解释php。

那么，php7_module是如何将数据传给php的解析器来解析php代码的呢？
答案是：sapi

用一张图来看apache、php、sapi三者之间的关系： 

![640.webp](https://pic.imgdb.cn/item/6119136a5132923bf82fb5d2.png)


从上面图中，我们看出了sapi就是这样的一个中间过程，sapi提供了一个和外部通信的接口，使得PHP可以和其他应用进行交互数据（apache，nginx等）。

php默认提供了很多种SAPI，常见的提供给apache和nginx的php7_module、CGI、FastCGI，给IIS的**ISAPI**，以及Shell的CLI。

httpd是Apache超文本传输协议(HTTP)服务器的主程序。被设计为一个独立运行的后台进程，它会建立一个处理请求的子进程或线程池。

所以，以上的apache调用php执行的过程如下：

apache -> httpd -> php7_module -> SAPI -> php
这种模式将php模块安装到apache中，每一次apache请求，都会产生一条进程，这个进程就完整的包括php的各种运算计算等操作。

在上图中，我们很清晰的可以看到，apache每接收一个请求，都会产生一个进程来连接php通过sapi来完成请求，可想而知，如果一旦用户过多，并发数过多，服务器就会承受不住了。而且，把php当做一个模块加载到apache中，出问题时很难定位是php的问题还是apache的问题。 

##### 第二种：mod_fastcgi模块（mod_cgi基本很少了）

fastcgi：http服务器与你的或其它机器上的程序进行“交谈”的一种工具，相当于一个程序接口。它可以接受来自web服务器的请求，解释输入信息，将处理后的结果返回给服务器等。mod_fastcgi就是在apache下支持fastcgi协议的模块。

使用fastcgi，最主要的优点是把应用和web server（apache）分离开来。这样允许把web server 和动态语言（php）运行在不同的主机上，以大规模扩展和改进安全性而不损失效率。

这样情况下，对于php-cgi程序，由于从apache中分离出来，就需要一个单独的工具来对这些进程进行管理，幸运的就是php-fpm的出现。

![120212_eaA7_1384334.png](https://pic.imgdb.cn/item/611938555132923bf8124a5a.png)

##### 总结：
1）mod_php是apache的内置php解释模块，使用prefork方式，不需要额外的进程来做通讯和应用解释，显然mod_php比mod_fastcgi这样方式性能要好得多

2）缺点是把应用和HTTP服务器绑定在了一起，当php模块出现问题可能会导致Apache一同挂掉

3）另外每个Apache进程都需要加载mod_php而不论这个请求是处理静态内容还是动态内容，这样导致浪费内存，效率下降，

4）php.ini文件的变更需要重新启动apache服务器才能生效，这使得无法进行平滑配置变更。


## 目前状况
随着技术的不断升级，单纯的Apache加php模块的方式已不再主流，而是替换为Apache加php_cgi，以及后来的php_fcgi和nginx加php-fpm的方法。

总结一下这个升级的过程： 
![643.webp](https://pic.imgdb.cn/item/61191c285132923bf85b618b.png)

<font color="red">如果要搭建一个高性能的PHP WEB服务器，目前最佳的方式是Apache/Nginx + FastCGI + PHP-FPM(+PHP-CGI)方式了。 </font>
