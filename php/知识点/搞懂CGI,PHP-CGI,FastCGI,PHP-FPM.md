
## 一、总览

在整个网站架构中，Web Server（如Apache、Nginx）只是内容的分发者。举个栗子，如果客户端请求的是 index.html，那么Web Server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。 

如果请求的是 index.php，根据配置文件，Web Server知道这个不是静态文件，需要去找 PHP 解析器（Web Application）来处理，那么他会把这个请求简单处理，然后交给PHP解析器。 

![6401.webp](https://pic.imgdb.cn/item/611914a45132923bf83576d9.png)

当Web Server收到index.php 这个请求后，会启动对应的 CGI 程序(PHP-CGI)，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以CGI规定的格式返回处理后的结果，退出进程，Web server再把结果返回给浏览器。这就是一个完整的动态PHP Web访问流程。

Web Server 一般指Apache、Nginx、IIS、Tomcat等服务器，Web Application 一般指PHP、Java、Asp.net等应用程序. 

## 二、CGI
### CGI是什么
通用网关接口（Common Gateway Interface/CGI）描述了客户端和服务器程序之间传输数据的一种标准，它的作用就是帮助服务器与语言通信，比如nginx和php的语言不通，因此需要一个沟通转换的过程，而CGI就是这个沟通的协议。CGI是 Web Server 与 Web Application 之间<font color="red">数据交换的一种协议</font>。

<font color="red">用来规范web服务器传输到php解释器中的数据类型以及数据格式</font>，包括URL、查询字符串、POST数据、HTTP header等，也就是为了保证web server传递过来的数据是标准格式的。同时php解释器完成任务后传递给web server也要遵循CGI协议返回给web服务器，最后web server接到这些信息再返回给浏览器。

CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php、perl、tcl等。

不同类型语言写的程序只要符合cgi标准，就能作为一个cgi程序与web服务器交互，早期的cgi大多都是c或c++编写的。

**一般说的CGI指的是用各种语言编写的能实现该功能的程序。**

要对php-cgi进程进行管理,需要使用特别的模块:比如Apache的mod_fcgid,IIS的PHP Manager。

而Nginx只负责反向代理/请求转发,不负责管理php-cgi进程,所以Nginx一般配合能够自行管理工作进程(子进程)的php-fpm使用.

**需要注意的是,php-fpm是一个独立的SAPI,其管理的不是php-cgi,也就是说php-fpm跟php-cgi无关,php-fpm内置php解释器,php-fpm的子进程是自己fork出来的,并不会调用php-cgi,你把系统中的php-cgi删了也不会影响到php-fpm服务的正常运行。**
 

### CGI的历史
早期的webserver只处理html等静态文件，但是随着技术的发展，出现了像php等动态语言。
webserver处理不了了，怎么办呢？那就交给php解释器来处理吧！

但是php解释器如何与webserver进行通信呢？

为了解决不同的语言解释器(如php、python解释器)与webserver的通信，于是出现了cgi协议。CGI 独立于任何语言的，只要你按照cgi协议去编写程序，CGI 程序可以用任何脚本语言或者是完全独立编程语言实现，只要这个语言可以在这个系统上运行，就能实现语言解释器与webwerver的通信。如**php-cgi程序**。

### CGI程序的工作原理
1. web 服务器收到客户端（浏览器）的请求Http Request，启动CGI程序，并通过环境变量、标准输入传递数据

2. cgi进程启动解析器、加载配置（如业务相关配置）、连接其它服务器（如数据库服务器）、逻辑处理等

3. cgi程将处理结果通过标准输出、标准错误，传递给web 服务器

4. web 服务器收到cgi返回的结果，构建Http Response返回给客户端，并杀死cgi进程

> web服务器与cgi通过环境变量、标准输入、标准输出、标准错误互相传递数据。

![191104434855332.png](https://pic.imgdb.cn/item/612af83e44eaada73982dc7a.png)

### 环境变量
GET请求，它将数据打包放置在环境变量QUERY_STRING中，CGI从环境变量QUERY_STRING中获取数据。常见的环境变量如下表所示：


| 环境变数 | 名称 |
| --- | --- |
| AUTH_TYPE |  存取认证类型。|
|CONTENT_LENGTH  | 由标准输入传递给CGI程序的数据长度，以bytes或字元数来计算。|
|  CONTENT_TYPE|请求的MIME类型。|
|GATEWAY_INTERFACE  | 服务器的CGI版本编号。 |
|  HTTP_ACCEPT| 浏览器能直接接收的Content-types, 可以有HTTP Accept header定义. |
| HTTP_USER_AGENT | 递交表单的浏览器的名称、版本 和其他平台性的附加信息。 |
| HTTP_REFERER | 递交表单的文本的 URL，不是所有的浏览器都发出这个信息，不要依赖它。|
| PATH_INFO | 传递给cgi程式的路径信息。|
| QUERY_STRING | 传递给CGI程式的请求参数，也就是用"?"隔开，添加在URL后面的字串。|
|REMOTE_ADDR  | client端的host名称。 |
| REMOTE_HOST | client端的IP位址。 |
| REMOTE_USER | client端送出来的使用者名称。 |
| REMOTE_METHOD | client端发出请求的方法（如get、post）。 |
| SCRIPT_NAME | CGI程式所在的虚拟路径，如/cgi-bin/echo。 |
| SERVER_NAME |  server的host名称或IP地址。|
|SERVER_PORT|收到request的server端口。|
|SERVER_PROTOCOL|所使用的通讯协定和版本编号。|
|SERVER_SOFTWARE|server程序的名称和版本。|

###标准输入
环境变量的大小是有一定的限制的，当需要传送的数据量大时，储存环境变量的空间可能会不足，造成数据接收不完全，甚至无法执行CGI程序。因此后来又发展出另外一种方法：POST，也就是利用I/O重新导向的技巧，让CGI程序可以由STDIN和STDOUT直接跟浏览器沟通。

当我们指定用这种方法传递请求的数据时，web 服务器收到数据后会先放在一块输入缓冲区中，并且将数据的大小记录在CONTENT_LENGTH这个环境变数，然后调用CGI程式并将CGI程序的STDIN指向这块缓冲区，于是我们就可以很顺利的通过STDIN和环境变数CONTENT_LENGTH得到所有的资料，再没有资料大小的限制了。

### CGI优点：
1 CGI的好处就是完全独立于任何服务器，仅仅是做为中间分子。提供接口给web服务器和web应用(如提nginx和php)。他们通过cgi搭线来完成数据传递。这样做的好处了尽量减少2个的关联，使他们2个变得更独立。

2 CGI对php.ini的配置很敏感，在开发和调试的时候相当方便

### CGI缺点：
1 <font color="red">高并发时的性能较差</font>
webserver每收到一个请求，都会去fork一个cgi进程，请求结束再kill掉这个进程。，也就是最为人诟病的**fork-and-execute**模式，这样一在大规模并发下，几乎是不可用的。 

2 传统的CGI接口方式安全性较差

## 三、FastCGI
### FastCGI是什么
FastCGI（Fast Common Gateway Interface）全称“快速通用网关接口”。同 CGI，也是一种通信协议，**是CGI的改进版本**，由CGI发展改进而来。

FastCGI致力于减少网页服务器与CGI程序之间互动的开销，从而使服务器可以同时处理更多的网页请求（提高并发访问）。fast-cgi每次处理完请求后，不会kill掉这个进程，而是保留这个进程，使这个进程可以一次处理多个请求。这样每次就不用重新fork一个进程了，大大提高了效率。这些进程由FastCGI进程管理器管理。

‘FastCGI进程管理器’并不是一个程序的名称，而是指遵循FastCGI协议的一类程序。

**同样的，一般说的FastCGI指的也是用各种语言编写的能实现该功能的程序。**

![191104459072505.png](https://pic.imgdb.cn/item/612afb5f44eaada739883f11.png)

### FastCGI工作原理
1. Web 服务器启动时载入初始化FastCGI执行环境 。 例如IIS ISAPI、apache mod_fastcgi、nginx ngx_http_fastcgi_module、lighttpd mod_fastcgi

2. FastCGI进程管理器自身初始化，启动多个CGI解释器进程并等待来自Web 服务器的连接。启动FastCGI进程时，可以配置以ip和UNIX 域socket两种方式启动。 

3. 当客户端请求到达Web 服务器时， Web 服务器将请求采用socket方式转发到 FastCGI主进程，FastCGI主进程选择并连接到一个CGI解释器。Web 服务器将CGI环境变量和标准输入发送到FastCGI子进程。

4. FastCGI子进程完成处理后将标准输出和错误信息从同一socket连接返回Web 服务器。当FastCGI子进程关闭连接时，请求便处理完成。

5. FastCGI子进程接着等待并处理来自Web 服务器的下一个连接。

![191104485636450.png](https://pic.imgdb.cn/item/612afb5f44eaada739883f11.png)

其他流程图
![6402.webp](https://pic.imgdb.cn/item/611919c65132923bf84eebc7.png)


### FastCGI对进程的管理方式
<font color="red">FastCGI程序会先启动一个master，解析配置环境，初始化执行环境，然后再启动多个worker</font>。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源，这就是fastcgi对进程的管理。（CGI程序和FastCGI程序，可以理解成遵循CGI协议和FastCGI协议编写的程序）

### 要对FastCGI进程进行管理,需要使用特别的模块
* Apache的mod_fcgid(题外话：php还可以作为apache的子模块运行)
* IIS的PHP Manager
* Nginx只负责反向代理/请求转发,不负责管理php-cgi进程,所以Nginx一般配合能够自行管理工作进程(子进程)的php-fpm使用.

### FastCGI的优点
1）FastCGI具有语言无关性，支持用大多数语言进行编写，对应的程序也支持大多数主流的web服务器。

FastCGI技术目前支持语言有：C/C++，Java，PHP，Perl，Tcl，Python，SmallTalk，Ruby等。
    
支持FastCGI技术的主流web服务器有：Apache，Nginx，lighttpd等

2）FastCGI程序的接口方式采用C/S结构，可以将web服务器和脚本解析服务器分开，独立于web服务器运行，提高web服务器的并发性能和安全性：

提高性能：这种方式支持多个web分发服务器和多个脚本解析服务器的分布式架构，同时可以在脚本解析服务器上启动一个或者多个脚本解析守护进程来处理动态请求，可以让web服务器专一地处理静态请求或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

提高安全性：API方式把应用程序的代码与核心的web服务器链接在一起，这时一个错误的API的应用程序可能会损坏其他应用程序或核心服务器，恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥，采用这种方式可以在很大程度上避免这个问题

3）FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。

4）FastCGI程序在修改php.ini配置时可以进行平滑重启加载新配置。

所有的配置加载都只在FastCGI进程启动时发生一次，每次修改php.ini配置文件，只需要重启FastCGI程序（php-fpm等）即可完成平滑加载新配置，已有的动态请求会继续处理，处理完成关闭进程，新来的请求使用新加载的配置和变量进行处理。

5）FAST-CGI是较新的标准，架构上和CGI大为不同，是用一个驻留内存的服务进程向网站服务器提供脚本服务。像是一个<font color="red">常驻</font>(long-live)型的CGI，维护的是一个进程池，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次（这是CGI最为人诟病的fork-and-execute 模式），速度和效率比CGI大为提高，是目前的主流部署方式。它的速度效率最少要比CGI 技术提高 5 倍以上。

6）支持分布式的部署， 即 FastCGI 程序可以在web 服务器以外的主机上执行。

### FastCGI的缺点
1）FastCGI的不足：因为是在内存中同时运行多进程，所以会比CGI方式消耗更多的服务器内存，每个PHP-CGI进程消耗7至25兆内存，在进行优化配置php-cgi进程池的数量时要注意系统内存，防止过量

## 四、PHP-CGI
使用php实现CGI协议的CGI<font color="red">接口程序</font>，是PHP的解释器，提供给web serve也就是http前端服务器的cgi协议接口程序，当每次接到http前端服务器的请求都会开启一个php-cgi进程进行处理，而且开启的php-cgi的过程中会先要重载配置，数据结构以及初始化运行环境。他自己本身只能解析请求，返回结果，不会进程管理。

使用如下命令可以启动PHP-CGI：
```sh
php-cgi -b 127.0.0.1:9000
```
这时候就可以修改nginx或者apache的解析socket ip 访问到127.0.0.1:9000，就可以正常解析了。

php-cgi的特点：

* php-cgi变更php.ini配置后需重启php-cgi才能让新的配置生效，不可以平滑重启

* 直接杀死php-cgi进程php就不能运行了

* php-cgi既支持CGI协议，也支持fast-CGI协议。



## 五、PHP-FPM
PHP-FPM(FastCGI Process Manager)。
是 PHP(Web Application)对 Web Server 提供的 FastCGI 协议的<font color="red">接口程序</font>。它额外还提供了相对智能一些任务管理(比如平滑过渡配置更改，内存和进程控制更有效)。

**PHP-FPM是FastCGI进程管理器。**前面说了，FastCGI进程管理器是一类程序，而PHP-FPM就属于这一类程序中的一个。

它不会像php-cgi一样每次连接都会重新开启一个进程，处理完请求又关闭这个进程，而是允许一个进程对多个连接进行处理，而不会立即关闭这个进程，而是会接着处理下一个连接。它的守护进程会平滑从新生成新的子进程。 

php-fpm会开启多个php-cgi程序，并且php-fpm常驻内存，每次web serve服务器发送连接过来的时候，php-fpm将连接信息分配给下面其中的一个子程序php-cgi进行处理，处理完毕这个php-cgi并不会关闭，而是继续等待下一个连接，这也是fast-cgi加速的原理，但是由于php-fpm是多进程的，而一个php-cgi基本消耗7-25M内存，因此如果连接过多就会导致内存消耗过大，引发一些问题，例如nginx里的502错误。


**注意**
* php54之前，php-fpm(第三方编译)是管理器，php-cgi是解释器
* php54之后，php-fpm(官方自带)，master 与 pool 模式。php-fpm 和 php-cgi 没有关系了。**php-fpm又是解释器，又是管理器。**
* php-fpm就是php-cgi的升级版（并非简单的在php-cgi的基础上升级，而且通过直接采用第三方代码的方式，实质上是用php-fpm“替换”了php-cgi，而不是简单的升级，但我们可以理解为升级),php-fpm比php-cgi高级很多。

* win不支持php-fpm，因为php-fpm是使用Linux的fork()来做的，所以win下面基本上还是使用php-cgi。
 
**1）PHP-FPM使用PHP编写的PHP-FastCGI管理器，管理对象是PHP-CGI程序，不能说php-fpm是fastcgi进程的管理器，因为前面说了fastcgi是个协议。**

在配置时使用--enable-fpm参数即可开启PHP-FPM

2）修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。

### php-fpm编译
./configure 的时候添加 --enable-fpm 选项，就能编译出php-fpm。

## 六、web server和Web Application的交互
### 静态页面
![20170404133415927.png](https://pic.imgdb.cn/item/612af6e844eaada73980999d.png)

### 普通cgi模式下的动态页面
![20170404153524167.png](https://pic.imgdb.cn/item/612af78f44eaada73981ac5b.png)

### php-fpm模式的动态页面
![20170404151830456.png](https://pic.imgdb.cn/item/612af70e44eaada73980d629.png)

 






