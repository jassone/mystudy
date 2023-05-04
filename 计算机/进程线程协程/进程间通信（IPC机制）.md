## IPC定义

IPC是Inter-Process Communication的缩写，含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。任何一个操作系统都需要有相应的IPC机制，比如Windows上可以通过剪贴板、管道和邮槽等来进行进程间通信，而Linux上可以通过命名共享内容、信号量等来进行线程间通信。

总共有以下一些方式：

- **管道**分为命名管道和无名管道，在内核中申请一块固定大小的缓冲区，程序拥有写入和读取的权利，都可以看成一种特殊的文件，具有固定的读端和写端，也可以使用普通的read、write 等函数。但是它不是普通的文件，并不属于其他任何文件系统，并且只存在于内存中；无名管道一般使用fork函数实现父子进程的通信，命名管道用于没有血缘关系的进程也可以进程间通信；面向字节流、自带同步互斥机制、半双工，单向通信，两个管道实现双向通信。
- **消息队列**，在内核中创建一队列，队列中每个元素是一个数据报，不同的进程可以通过句柄去访问这个队列；消息队列独立于发送与接收进程，可以通过顺序和消息类型读取，也可以fifo读取；消息队列可实现双向通信。
- **信号量** ， 在内核中创建一个信号量集合（本质是个数组），数组的元素（信号量）都是1，使用P操作进行-1，使用V操作+1，通过对临界资源进行保护实现多进程的同步
- **共享内存**，将同一块物理内存一块映射到不同的进程的虚拟地址空间中，实现不同进程间对同一资源的共享。目前最快的IPC形式，不用从用户态到内核态的频繁切换和拷贝数据，直接从内存中读取就可以，共享内存是临界资源，所以需要操作时必须要保证原子性。使用信号量或者互斥锁都可以。
- **socket**是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口，把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据。socket起源于UNIX，在Unix一切皆文件哲学的思想下，socket是一种”打开—读/写—关闭”模式的实现，服务器和客户端各自维护一个”文件”，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。是一种可以**网间通信**的方式。

## 一、socket方式
### 1、tcp socket和UNIX Domain Socket

UNIX Domain Socket可用于两个没有亲缘关系的进程,是目前广泛使用的IPC机制,比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的.这种通信方式是发生在系统内核里而不会在网络里传播。UNIX Domain Socket和长连接都能避免频繁创建TCP短连接而导致TIME_WAIT连接过多的问题。

一般服务都提供两种交互方式：tcp socket与unix domain socket，两者区别：

如下图所示，对于进程间通讯的两个程序，unix domain socket的流程不会走到TCP 那层，直接以文件形式，**以stream socket通讯**。如果是TCP socket,则需要走到IP层。对于非同一台服务器上，TCP socket走的就更多了。

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