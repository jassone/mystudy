## PHP-FPM(Fast CGI Process Manager)
是一个PHPFast CGI管理器。全称是PHP FastCGI Process Manager。
通俗来讲，就是用来管理启动一个master进程和多个worker进程的程序。即提供了进程管理的功能。php-fpm在php5.3以后不再作为第三方的模块而是集成到了php中。

**FastCGI进程包含master进程和worker进程两种进程。master进程只有一个，只负责监听端口，接收apache/Nginx的请求，而worker进程则一般有多个(可配置)，每个进程内部都嵌入了一个PHP解释器(php-cgi)，是PHP代码真正执行的地方。** 

### 使用
1 启动

```sh
php-fpm
```

* php 5.3.3 以后的php-fpm 不再支持 php-fpm start|stop|reload
* 不能关闭窗口，否则则关闭服务

2 其他
master进程可以理解以下信号

* INT, TERM 立刻终止
* QUIT 平滑终止
* USR1 重新打开日志文件
* USR2 平滑重载所有worker进程并重新载入配置和二进制模块

先查看php-fpm的master进程号

```sh
# ps aux | grep php-fpm | grep master | grep -v grep
root     13225  0.0  0.0 204820  7508 ?        Ss   09:37   0:01 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
You have new mail in /var/spool/mail/root
```


重启php-fpm:
```sh
kill -USR2 13225  
```

上面方案一般是没有生成php-fpm.pid文件时使用，如果要生成php-fpm.pid，使用下面这种方案：

上面master进程可以看到，matster使用的是/usr/local/php/etc/php-fpm.conf这个配置文件，cat /usr/local/php/etc/php-fpm.conf 发现：

```ini
[global]
; Pid file
; Note: the default prefix is /usr/local/php/var
; Default Value: none
;pid = run/php-fpm.pid
```
pid文件路径应该位于/usr/local/php/var/run/php-fpm.pid，
由于注释掉，所以没有生成，我们把注释去除，再kill -USR2 42891 重启php-fpm,便会生成pid文件，下次就可以使用以下命令重启,关闭php-fpm了。

```sh
php-fpm 关闭：
kill -INT 'cat /usr/local/php/var/run/php-fpm.pid'
php-fpm 重启：
kill -USR2 'cat /usr/local/php/var/run/php-fpm.pid'
```


### php-fpm 命令

```sh
测试php-fpm配置文件
➜  ~ git:(master) ✗ php-fpm -t
[18-Aug-2021 18:05:19] NOTICE: configuration file /usr/local/etc/php/7.4/php-fpm.conf test is successful
```

### 配置
/usr/local/etc/php/7.4（本人）
├── php-fpm.conf
├── php-fpm.d
│ └── www.conf
├── php.ini

##### <font color="red">pm.max_children</font>

确定了php-fpm的处理能力，原则上是越多越好，但这个是在内存足够大的前提下，每开启一个php-fpm进程要占用近30M左右的内存。如果请求访问较多，那么可能会出现502，504错误。**对于502错误来说，属于繁忙进程而造成的，对于504来说，就是客户发送的请求在限定的时间内没有得到相应，过多的请求导致“504 Gateway Time-out”。**这里也有可能是服务器带宽问题。

##### <font color="red">request_terminate_timeout</font>

它决定php-fpm进程的连接/发送和读取的时间（单个请求的超时中止时间），如果设置过小很容易出现"502 Bad Gateway" 和 “504 Gateway Time-out”，默认为0，就是说没有启用，不加限制，但是这种设置前提是你的php-fpm足够健康，这个需要根据实际情况加以限定。

php.ini里面max_execution_time 可以设置 PHP 脚本的最大执行时间，但是，在 php-cgi(php-fpm)中，该参数不会起效。真正能够控制 PHP 脚本最大执行时间的是 php-fpm.conf配置文件中的request_terminate_timeout参数。

##### <font color="red">listen.backlog = -1</font>
backlog是完成三次握手即处于established状态的用户连接队列的长度,就通过listen第二个参数指定.

backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。

有时候机子性能差是由于php-fpm backlog参数设置为-1，导致fpm没能及时取出完成连接队列的socket，出现SYN 超时，最终导致压不上去，表现出性能差。

所以安装php-fpm时backlog一定要重新设置，不能用fpm默认配置的-1 ，可以根据机器的并发量来设置，建议设置在1024以上，最好是2的幂值（因为内核会调整成2的n次幂）。

##### request_slowlog_timeout = 10s
当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'

##### slowlog = log/$pool.log.slow
慢请求的记录日志,配合request_slowlog_timeout使用
 。

##### pm相（三种运行模式）
* pm = static

    php-fpm在pm = static配置下工作进程常驻后台,也就是如果你配置了5个工作进程pm.max_children = 5,那php-fpm服务启动时就会自动fork出5个子进程并常驻后台,不会在请求处理结束后退出,也不会在空闲时退出.如果你在php脚本中使用了数据库持久连接,这时这5个工作进程还会建立并维持5个到数据库的持久连接,实现在处理多个请求的时候重用数据库连接资源,避免每个请求都建立/释放一次数据库连接.持久连接还能做到超时自动重连,对php-fpm里的脚本来说是完全透明的,脚本只需在启动时指明使用持久连接即可.

    对于专用服务器,可以设置为static。

    pm.max_children：静态方式下，子进程最大数

* pm = dynamic

    php-fpm在pm = dynamic配置下工作进程【部分】常驻后台,也就是维持一定数量的常驻进程,服务繁忙时fork出更多的进程,服务闲置时自动关掉一些进程,把内存资源归还给操作系统.虚拟主机提供商应该是比较喜欢这种方式的.总而言之,PHP-FPM这种运行模式类似于Apache的prefork MPM,能静能动的多进程网络服务.
    
    pm.max_children：动态方式下，子进程最大数

    pm.start_servers：动态方式下，启动时的进程数

    pm.min_spare_servers：动态方式下，保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程

    pm.max_spare_servers:动态方式下，保证空闲进程数最大值，如果空闲进程大于此值，此进行清理
    
* pm = ondemand
    启动时不会创建子进程，当新的请求到达时才创建
    
    pm.max_children：子进程最大数
    
    pm.process_idle_timeout:子进程的空闲超时时间，如果超时时间到没有新的请求可以服务，则会被杀死。

##### pm.max_requests = 1000
设置每个子进程重启之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 '0' 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0。上面配置相当于一个 PHP-CGI 进程处理的请求数累积到 1000 个后，自动重启该进程。


##### pid = run/php-fpm.pid
pid设置，默认在安装目录中的var/run/php-fpm.pid，用于重启和关闭php-fpm

##### error_log = log/php-fpm.log
错误日志，默认在安装目录中的var/log/php-fpm.log
 
##### log_level = notice
错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.
 

##### emergency_restart_threshold = 60; emergency_restart_interval = 60s
表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。


##### process_control_timeout = 0
设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0.


##### daemonize = yes
后台执行fpm,默认值为yes，如果为了调试可以改为no。**在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置。**


##### listen = 127.0.0.1:9000
fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port', '/path/to/unix/socket'. 每个进程池都需要设置.


##### listen.allowed_clients = 127.0.0.1
允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接

##### listen.owner = www
##### listen.group = www
启动进程的帐户和组

##### listen.mode = 0666
unix socket设置选项，如果使用tcp方式访问，这里注释即可。

##### unix socket，如果使用tcp方式访问，这里注释即可。

##### user = www;group = www
启动进程的帐户和组


#####   配置详见wiki
  https://www.php.net/manual/zh/install.fpm.configuration.php
  
  
### 优化
##### php-fpm优化参数
pm、pm.max_children、pm.start_servers、pm.min_spare_servers、pm.max_spare_servers。
pm.max_requests

##### 配置优化demo
1 配置运行方式和进程数

对于我们的服务器，选择哪种执行方式比较好呢？事实上，跟Apache一样，运行的PHP程序在执行完成后，或多或少会有内存泄露的问题。
这也是为什么开始的时候一个php-fpm进程只占用3M左右内存，运行一段时间后就会上升到20-30M的原因了。

对于内存大的服务器（比如8G以上）来说，指定静态的max_children实际上更为妥当，因为这样不需要进行额外的进程数目控制，会提高效率。

因为频繁开关php-fpm进程也会有时滞，所以内存够大的情况下开静态效果会更好。数量也可以根据 内存/30M 得到，比如8GB内存可以设置为100，那么php-fpm耗费的内存就能控制在 2G-3G的样子。如果内存稍微小点，比如1G，那么指定静态的进程数量更加有利于服务器的稳定。

这样可以保证php-fpm只获取够用的内存，将不多的内存分配给其他应用去使用，会使系统的运行更加畅通。

对于小内存的服务器来说，比如256M内存的VPS，即使按照一个20M的内存量来算，10个php-cgi进程就将耗掉200M内存，那系统的崩溃就应该很正常了。

因此应该尽量地控制php-fpm进程的数量，大体明确其他应用占用的内存后，给它指定一个静态的小数量，会让系统更加平稳一些。或者使用动态方式，因为动态方式会结束掉多余的进程，可以回收释放一些内存，所以推荐在内存较少的服务器或VPS上使用。具体最大数量根据 内存/20M 得到。

比如说512M的VPS，建议pm.max_spare_servers设置为20。至于pm.min_spare_servers，则建议根据服务器的负载情况来设置，比如服务器上只是部署php环境的话，比较合适的值在5~10之间。

设置每个子进程重生之前服务的请求数. 对于可能存在内存泄漏的第三方模块来说是非常有用的. 如果设置为 ’0′ 则一直接受请求. 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 默认值: 0.
这段配置的意思是，当一个 PHP-CGI 进程处理的请求数累积到 500 个后，自动重启该进程。

2 pm.max_requests配置

pm.max_requests是为了重启服务，但是为什么要重启进程呢？

一般在项目中，我们多多少少都会用到一些 PHP 的第三方库，这些第三方库经常存在内存泄漏问题，如果不定期重启 PHP-CGI 进程，势必造成内存使用量不断增长。因此 PHP-FPM 作为 PHP-CGI 的管理器，提供了这么一项监控功能，对请求达到指定次数的 PHP-CGI 进程进行重启，保证内存使用量不增长。

**正是因为这个机制，在高并发的站点中，经常导致 502 错误，我猜测原因是 PHP-FPM 对从 NGINX 过来的请求队列没处理好。**

目前我们的解决方法是，把这个值尽量设置大些，尽可能减少 PHP-CGI 重新 SPAWN 的次数，同时也能提高总体性能。在我们自己实际的生产环境中发现，内存泄漏并不明显，因此我们将这个值设置得非常大（204800）。大家要根据自己的实际情况设置这个值，不能盲目地加大。


  