## PHP-FPM+Nginx通信原理

## 一、几个概念

### PHP-FPM
PHP-FPM的全称是PHP FastCGI Process Manager，PHP-FPM是FastCGI的实现，并提供了进程管理的功能。 

### Nginx
Nginx (“engine x”) 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。这里介绍一下什么是正向代理和反向代理，这个对于我们理解Nginx很重要。

### 正向代理

以我们访问国外的网站为例，比如访问Google、Facebook。我们需要借助vpn才能访问，我们借助vpn访问国外的网站，其实就是个正向代理的过程，上图： 

![gaitubao_640_png.png](https://pic.imgdb.cn/item/61243ab944eaada73948fded.png)

vpn对于Users来说，是可以感知到的（因为用户需要配置连接），vpn对于google服务器来说，是不可感知的(google服务器只知道有http请求过来)。所以，对于用户来说可以感知到，而对于服务器来说感知不到服务器，就是正向代理服务器（vpn） 

### 反向代理
拿Nginx作为反向代理服务器实现负载均衡来举例，假设此时我们访问百度，看图： 

![gaitubao_640 (1)_png.png](https://pic.imgdb.cn/item/61243ae544eaada739492cc1.png)

当用户访问百度时，所有的请求会到达一个反向代理服务器，这个反向代理服务器会将请求分发给后边的某一台服务器去处理我们的请求。此时，这个代理服务器其实对用户来说是不可感知的，用户感知到的是百度的服务器给自己返回了结果，并不知道代理服务器的存在。也就是说，对于用户来说不可感知，对于服务器来说是可以感知的，就叫反向代理服务器（Nginx） 

## 二、PHP-FPM+Nginx通信

nginx 不能直接执行外部可执行程序，并且cgi是接收到请求时才会启动cgi进程，不像fastcgi会在一开始就启动好，这样nginx天生是不支持 cgi 的。nginx 虽然不支持cgi，但它支持 fastCGI。而Nginx不负责管理php-cgi进程,所以Nginx一般配合能够自行管理工作进程(子进程)的php-fpm使用。

FastCGI致力于减少Web服务器与CGI程序之间互动的开销，从而使服务器可以同时处理更多的Web请求。与CGI这种为每个请求创建一个新的进程不同，FastCGI使用持续的进程来处理一连串的请求，这些进程由FastCGI进程管理器管理，而不是web服务器。 

通过图来方便我们理解PHP-FPM和Nginx的通信 

![gaitubao_640 (2)_png.png](https://pic.imgdb.cn/item/61243b1b44eaada739496752.png)

1、当Nginx收到http请求（动态请求），它会初始化FastCGI环境。（如果是Apache服务器，则初始化mode_fastcgi模块、如果是Nginx服务器则初始化ngx_http_fastcgi_module）

2、我们在配置nginx解析php请求时，一般会有这样一行配置： 

```ini
fastcgi_pass   127.0.0.1:9000;
```
或者长这样： 
```ini
fastcgi_pass  unix:/tmp/php-cgi.sock;
```

它其实是Nginx和PHP-FPM一个通信载体（或者说通信方式），目的是为了让Nginx知道，收到动态请求之后该往哪儿发。（关于这两种配置的区别，后边会专门介绍）

3、Nginx将请求采用socket的方式转给FastCGI主进程

4、FastCGI主进程选择一个空闲的worker进程连接，然后Nginx将CGI环境变量和标准输入发送该worker进程（php-cgi）

5、worker进程完成处理后将标准输出和错误信息从同一socket连接返回给Nginx。

6、worker进程关闭连接，等待下一个连接 


### 不从配置的角度，再描述一下PHP和Nginx的通信 

1、**Nginx也是有master和worker进程**，worker进程直接处理每一个网络请求

2、其实在Nginx+PHP的架构里边，php可以看做是一个cgi程序的角色，因此出现了php-fpm进程管理器来处理这些php请求。php-fpm和nginx一样，也会监听端口（通过nginx.con里的配置我们知道，nginx默认监听8080端口，php-fpm默认监听9000端口），并且有master和worker进程，worker负责处理每一个php请求。

3、关于fastcgi：fastcgi是一个协议。市面上有多种实现了fastcgi协议的进程管理器，php-fpm就是其中的一种。php-fpm作为一种fastcgi进程管理服务，会监听端口，一般默认监听9000端口，并且是监听本机，也就是只接收来自本机的端口请求。

4、当需要处理php请求时，nginx的worker进程会将请求移交给php-fpm的worker进程进行处理，也就是最开头所说的nginx调用了php，其实严格的讲是nginx间接调用php（反向代理的方式）。

##### fastcgi_pass

Nginx和PHP-FPM的进程间通信有两种方式,一种是TCP Socket(**默认监听方式**),一种是Unix Socket

Tcp Socket方式是IP加端口,可以跨服务器.而UNIX Socket不经过网络,只能用于Nginx跟PHP-FPM都在同一服务器的场景，用哪种取决于你的PHP-FPM配置

Tcp Socket方式：

```ini
nginx.conf中配置：fastcgi_pass 127.0.0.1:9000;
php-fpm.conf中配置：listen=127.0.0.1:9000;
```

Unix Domain Socket方式:
```ini
nginx.conf中配置：fastcgi_pass unix:/tmp/php-fpm.sock;
php-fpm中配置：listen = /tmp/php-fpm.sock；
(php-fpm.sock是一个文件,由php-fpm生成)
```

两种通信配置方式，Nginx和PHP-FPM的通信过程如下：

Tcp Socket：
```
Nginx <=> socket <=> TCP/IP <=> socket <=> PHP-FPM
```
上边画Nginx和PHP-FPM通信的图时就是这种方式，这种情况是Nginx和PHP-FPM在同一台机器上。

看一下Nginx和PHP-FPM不在同一台机器上的情况： 
```
Nginx <=> socket <=> TCP/IP <=> 物理层 <=> 路由器 <=> 物理层 <=> TCP/IP <=> socket <=> PHP-FPM
```

Unix Socket： 
```
Nginx <=> socket <=> PHP-FPM
```

php5.3之后的版本，php-fpm不会生成php-fpm.sock，需要将listen的地址配置配置为UNIX Socket模式，同时保证这个路径已经存在，这样在启动./php-fpm的时候，会在对应路径上自动生成php-fpm.sock。

##### fastcgi_params

在nginx中有很多的fasgcgi_*的配置，更多的配置可以在nginx.conf的同级目录中看到，看一下里边的内容： 

![gaitubao_640 (3)_png.png](https://pic.imgdb.cn/item/61243dfc44eaada7394c6928.png)

这里边的内容都会被传递给PHP-FPM所管理的fastcgi进程。为什么会传递这些呢？相信大家都用过$_SERVER这个全局变量，这里边的值就是从此配置中拿到的。我们可以在fastcgi_params中看到REMOTE_ADDR，就是客户端地址，PHP之所以能拿到客户端信息，就是因为Nginx的配置中的fastcgi_params。

更多ngx_http_fastcgi_module模块的内容，看官方文档：
http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html


##### php-fpm.conf配置 
熟悉php-fpm的配置，能帮助我们优化服务器性能 

##### PHP-FPM进程池 

php-fpm.conf中默认配置了一个进程池，我们可以打开我们的php-fpm.conf看一下，下边是我的：

![gaitubao_640 (4)_png.png](https://pic.imgdb.cn/item/61250f6044eaada7391b5b70.png)

现在我们执行一下:ps -aux|grep php-fpm

![gaitubao_640 (5)_png.png](https://pic.imgdb.cn/item/61250f7b44eaada7391b9bcb.png)

会看见有一个master，10个worker进程，和我们配置的一样（www为进程池名） 

想配置多个，这样做即可：

![gaitubao_640 (6)_png.png](https://pic.imgdb.cn/item/61250fc744eaada7391c44eb.png)

在nginx中fastcgi_pass这个地方配置使用哪个进程池即可 

### nginx和php交互流程

##### 1 使用php-fpm方式---重要重要重要

```
www.example.com        |

       |

     Nginx        |

       |路由到www.example.com/index.php        |

       |加载nginx的fast-cgi模块        |

       |fast-cgi监听127.0.0.1:9000地址        |

       |www.example.com/index.php请求到达127.0.0.1:9000

       |

       |php-fpm 监听127.0.0.1:9000

       |

       |php-fpm 接收到请求，启用worker进程处理请求        |

       |php-fpm 处理完请求，返回给nginx        |

       |nginx将结果通过http返回给浏览器
```