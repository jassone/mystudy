
## 一、总览

在整个网站架构中，Web Server（如Apache、Nginx）只是内容的分发者。举个栗子，如果客户端请求的是 index.html，那么Web Server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。 
 
如果请求的是 index.php，根据配置文件，Web Server知道这个不是静态文件，需要去找 PHP 解析器来处理，那么他会把这个请求简单处理，然后交给PHP解析器。 

![6401.webp](https://pic.imgdb.cn/item/611914a45132923bf83576d9.png)

当Web Server收到index.php 这个请求后，会启动对应的 CGI 程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以CGI规定的格式返回处理后的结果，退出进程，Web server再把结果返回给浏览器。这就是一个完整的动态PHP Web访问流程。

* <font color="red">CGI</font>：是 Web Server 与 Web Application 之间<font color="red">数据交换的一种协议</font>。

* <font color="red">FastCGI</font>：同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。

* <font color="red">PHP-CGI</font>：是 PHP （Web Application）对 Web Server 提供的 CGI 协议的接口程序。

* <font color="red">PHP-FPM</font>：(PHP FastCGI Process Manager, PHP FastCGI进程管理器）是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理。

（Web Server 一般指Apache、Nginx、IIS、Tomcat等服务器，Web Application 一般指PHP、Java、Asp.net等应用程序） 


## 二、概念

### 1、CGI
#### CGI是什么
1）CGI（Common Gateway Interface）全称是“通用网关接口”，WEB 服务器与PHP应用进行“交谈”的一种工具，其程序须运行在网络服务器上。

<font color="red">用来规范web服务器传输到php解释器中的数据类型以及数据格式</font>，包括URL、查询字符串、POST数据、HTTP header等，也就是为了保证web server传递过来的数据是标准格式的。

2）CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php、perl、tcl等。

不同类型语言写的程序只要符合cgi标准，就能作为一个cgi程序与web服务器交互，早期的cgi大多都是c或c++编写的。

3) 一般说的CGI指的是用各种语言编写的能实现该功能的程序。

#### CGI程序的工作原理
1）每次当web server收到index.php这种类型的动态请求后，会启动对应的CGI程序（PHP的解析器）；
2）PHP解析器会解析php.ini配置文件，初始化运行环境，然后处理请求，处理完成后将数据按照CGI规定的格式返回给web server然后退出进程；
3）最后web server再把结果返回给浏览器。

#### CGI优点：
1 CGI的好处就是完全独立于任何服务器，仅仅是做为中间分子。提供接口给web服务器和web应用(如提nginx和php)。他们通过cgi搭线来完成数据传递。这样做的好处了尽量减少2个的关联，使他们2个变得更独立。
2 CGI对php.ini的配置很敏感，在开发和调试的时候相当方便

#### CGI缺点：
1 <font color="red">高并发时的性能较差</font>
每一次web请求都会有启动和退出过程，也就是最为人诟病的**fork-and-execute**模式，这样一在大规模并发下，几乎是不可用的。 

2 传统的CGI接口方式安全性较差

### 2、FastCGI
#### FastCGI是什么
1）FastCGI（Fast Common Gateway Interface）全称“快速通用网关接口”。是通用网关接口（CGI）的增强版本，由CGI发展改进而来。

主要用来提高CGI程序性能，类似于CGI，FastCGI也是一种让交互程序与Web服务器通信的协议

2）FastCGI致力于减少网页服务器与CGI程序之间互动的开销，从而使服务器可以同时处理更多的网页请求（提高并发访问）。

3）同样的，一般说的FastCGI指的也是用各种语言编写的能实现该功能的程序。

#### FastCGI工作原理
1）Web Server启动同时，加载FastCGI进程管理器（nginx的php-fpm或者IIS的ISAPI或Apache的Module)

2）FastCGI进程管理器读取php.ini配置文件，对自身进行初始化，启动多个CGI解释器进程(php-cgi)，等待来自Web Server的连接。

3）当Web Server接收到客户端请求时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server会将相关环境变量和标准输入发送到FastCGI子进程php-cgi进行处理

4）FastCGI子进程完成处理后将数据按照CGI规定的格式返回给Web Server，然后关闭FastCGI子进程或者等待下一次请求。

![6402.webp](https://pic.imgdb.cn/item/611919c65132923bf84eebc7.png)
 

#### FastCGI对进程的管理方式
<font color="red">FastCGI程序会先启动一个master，解析配置环境，初始化执行环境，然后再启动多个worker</font>。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源，这就是fastcgi对进程的管理。（CGI程序和FastCGI程序，可以理解成遵循CGI协议和FastCGI协议编写的程序）

#### FastCGI的特点
1）FastCGI具有语言无关性，支持用大多数语言进行编写，对应的程序也支持大多数主流的web服务器。

     
FastCGI技术目前支持语言有：C/C++，Java，PHP，Perl，Tcl，Python，SmallTalk，Ruby等。
    
支持FastCGI技术的主流web服务器有：Apache，Nginx，lighttpd等

2）FastCGI程序的接口方式采用C/S结构，可以将web服务器和脚本解析服务器分开，独立于web服务器运行，提高web服务器的并发性能和安全性：

 
提高性能：这种方式支持多个web分发服务器和多个脚本解析服务器的分布式架构，同时可以在脚本解析服务器上启动一个或者多个脚本解析守护进程来处理动态请求，可以让web服务器专一地处理静态请求或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

提高安全性：API方式把应用程序的代码与核心的web服务器链接在一起，这时一个错误的API的应用程序可能会损坏其他应用程序或核心服务器，恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥，采用这种方式可以在很大程度上避免这个问题
 
3）FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变

4）FastCGI程序在修改php.ini配置时可以进行平滑重启加载新配置


所有的配置加载都只在FastCGI进程启动时发生一次，每次修改php.ini配置文件，只需要重启FastCGI程序（php-fpm等）即可完成平滑加载新配置，已有的动态请求会继续处理，处理完成关闭进程，新来的请求使用新加载的配置和变量进行处理

5）FAST-CGI是较新的标准，架构上和CGI大为不同，是用一个驻留内存的服务进程向网站服务器提供脚本服务。像是一个<font color="red">常驻</font>(long-live)型的CGI，维护的是一个进程池，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次（这是CGI最为人诟病的fork-and-execute 模式），速度和效率比CGI大为提高，是目前的主流部署方式。

6）FastCGI的不足：因为是在内存中同时运行多进程，所以会比CGI方式消耗更多的服务器内存，每个PHP-CGI进程消耗7至25兆内存，在进行优化配置php-cgi进程池的数量时要注意系统内存，防止过量

### 3、PHP-CGI
使用php实现CGI协议的CGI程序，是PHP的解释器，他自己本身只能解析请求，返回结果，不会进程管理。

使用如下命令可以启动PHP-CGI：
```
php-cgi -b 127.0.0.1:9000
```

php-cgi的特点：

1）php-cgi变更php.ini配置后需重启php-cgi才能让新的配置生效，不可以平滑重启

2）直接杀死php-cgi进程php就不能运行了


### 4、PHP-FPM
PHP-FPM(FastCGI Process Manager)。

<font color="red">fastcgi是一个协议，php-fpm实现了这个协议。</font>
 
针对PHP-CGI的不足，PHP-FPM应运而生，它的守护进程会平滑从新生成新的子进程。 

1）PHP-FPM使用PHP编写的PHP-FastCGI管理器，管理对象是PHP-CGI程序，不能说php-fpm是fastcgi进程的管理器，因为前面说了fastcgi是个协议
 
在配置时使用--enable-fpm参数即可开启PHP-FPM

2）修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。

##### php-fpm编译
./configure 的时候添加 --enable-fpm 选项，就能编译出php-fpm。


  