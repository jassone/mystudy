## IPC定义
IPC是Inter-Process Communication的缩写，含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。任何一个操作系统都需要有相应的IPC机制，比如Windows上可以通过剪贴板、管道和邮槽等来进行进程间通信，而Linux上可以通过命名共享内容、信号量等来进行线程间通信。

## 一、几种实现方式
### 1、UNIX Domain Socket

UNIX Domain Socket可用于两个没有亲缘关系的进程,是目前广泛使用的IPC机制,比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的.这种通信方式是发生在系统内核里而不会在网络里传播.UNIX Domain Socket和长连接都能避免频繁创建TCP短连接而导致TIME_WAIT连接过多的问题.对于进程间通讯的两个程序,UNIX Domain Socket的流程不会走到TCP那层,直接以文件形式,以stream socket通讯.如果是TCP Socket,则需要走到IP层,对于非同一台服务器上,TCP Socket走的就更多了.

一般服务都提供两种交互方式：tcp socket与unix domain socket，两者区别：

如下图所示，对于进程间通讯的两个程序，unix domain socket的流程不会走到TCP 那层，直接以文件形式，以stream socket通讯。如果是TCP socket,则需要走到IP层。对于非同一台服务器上，TCP socket走的就更多了。

![2019062715392898.png](https://pic.imgdb.cn/item/61258bf044eaada739b40987.png)

举例(nginx和php-fpm交互)
UNIX Domain Socket:

```
Nginx <=> socket <=> PHP-FPM
```

TCP Socket(本地回环):
```
Nginx <=> socket <=> TCP/IP <=> socket <=> PHP-FPM
```

TCP Socket(Nginx和PHP-FPM位于不同服务器,不过也只能这样):
```
Nginx <=> socket <=> TCP/IP <=> 物理层 <=> 路由器 <=> 物理层 <=> TCP/IP <=> socket <=> PHP-FPM
```

其他像mysql命令行客户端连接mysqld服务也类似有这两种方式:

使用Unix Socket连接(默认):
```sh
mysql -uroot -p --protocol=socket --socket=/tmp/mysql.sock
```
使用TCP连接:
```sh
mysql -uroot -p --protocol=tcp --host=127.0.0.1 --port=3306
```

## 二、相关wiki
* 进程间的通信方式：https://www.jianshu.com/p/e0ea38ec4227