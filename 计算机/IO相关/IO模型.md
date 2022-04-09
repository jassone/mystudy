## I/O模型
## 一、先了解几个概念
### 1、流是什么
流是可以进行I/o操作的内核对象，比如文件、管道、套接字等。
流的入口：文件描述符(fd)，socket描述符(fd)等。

IO操作即对流进行读写。

![20220227095546.jpg](https://pic.imgdb.cn/item/621ada2a2ab3f51d91d8fd17.jpg)

### 2、I/0 操作的两部分
* 数据准备，将数据加载到内核缓存（数据加载到操作系统）
* 将内核缓存中的数据加载到用户缓存（从操作系统复制到应用中）

### 3、同步和异步
**同步和异步的概念描述的是用户线程与内核的交互方式。**

**同步和异步是针对应用程序来说的，关注的是程序中间的协作关系。**
异步是IO最理想的模型：cpu的原则是有必要了才会参与，也不浪费也不怠慢。

### 4、阻塞和非阻塞
**阻塞和非阻塞的概念描述的是用户线程调用内核IO操作的方式。**

**阻塞和非阻塞关注的是单个进程的执行状态。**

阻塞，可以看做是进程被休息了，cpu处理其他事情了。

非阻塞，可以理解为，将大的整片时间的阻塞分成N多的小的阻塞，所以进程不断地有机会被cpu光顾，理论上可以做点其他事情，但是cpu会很大概率因socket没有数据而空转，浪费资源。

阻塞等待：比如只有读，没有写，则会阻塞等待。
非阻塞，忙轮询：比如while(1){},占用cpu、系统资源。

* 区分阻塞和非阻塞：看read/write函数调用是否立即返回，是则是非阻塞。
* 区分同步和异步IO:看read/write操作是应用程序完成，还是操作系统完成，再通知应用程序。若是操作系统完成则是异步IO.

推荐优先使用阻塞等待模式。

## 二、linux操作系统四种IO模型
### 1、同步阻塞IO---同步IO
linux系统的read和write函数，在调用的时候会被阻塞，直到数据读取完成，或者写入成功。

### 2、同步非阻塞IO---同步IO
默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK。即和同步阻塞IO的API是一样的，只是打开fd的时候带有O_NONBLOCK参数，于是当调用read和write函数的时候，如果没有准备好数据，会立即返回，不会阻塞，然后让应用程序不断地去轮询。

### 3、IO多路复用(IO Multiplexing)---同步IO
也叫同步非阻塞IO，经典的Reactor设计模式。

前面两种IO都只能用于简单的客户端开发，但对于服务器来说，需要处理很多的fd（连接数可以达到几十万或者百万）。如果使用同步阻塞IO，要处理那么多的fd需要开非常多的线程，每个线程处理一个fd；如果使用同步非阻塞IO，用应用程序轮询这么大规模的fd。

两种办法都不行，于是就有了IO多路复用，这也是最常见的一种。

linux中，有三种IO多路复用的办法：select，poll，**epoll(用的最多，主流)**

##### a) select
```c
int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```
##### b) poll

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
struct pollfd{
	int fd;			//文件描述符
	short events;	//等待的事件
	short revents;	//实际发生的事件
}

```
通过上面函数会发现，select、poll每次调用都需要把应用程序fd的数组传进去，这个fd的数组每次都要在用户态和内核态之间传递，影响效率。

##### c) epoll
//该函数生成一个epoll专用的文件描述符(epoll的句柄)，size用来告诉内核监听的数目一共有多少。但不是准确的数字。
// 只是告诉内核，计划监听多少个fd。实际通过epoll_ctl添加的fd数目可能大于这个值。
```c
int epoll_create（int size）
```

//该函数用于控制某个文件描述符上的事件，将一个fd增/删/改到epfd里，对应的事件也即读/写
```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

//该函数用于轮询I/O事件的发生；
//多长时间去轮询一次， timeout=-1表示如果IO没有数据，一直阻塞。 正数表示超时时间
//返回几个有数据可读或可写的IO
```c
int epoll_wait(int epfd,struct epoll_event   *events,int maxevents,int timeout)
```

##### epoll的LT和ET模式
* LT：水平触发或条件触发
    读缓冲区只要不为空，就会一直触发读事件；
    写缓冲只要不满，就会一直触发写实际；

* ET：边缘触发或状态触发
    读缓冲区的状态，从空转为非空的时候触发一次(可读)；
    写缓冲区的状态，从满转为非满的时候触发一次(可写)

注意：
* 对于LT模式，要避免‘写的死循环’问题，即写缓冲区为满的概率很小，所以当用户注册了写事件却没有数据要写时，它会一直触发，因此在LT模式下写完数据一定要取消写事件。

* 对于ET模式，要避免‘short read’问题：比如用户收到100个字节，它触发一次，但用户只读到50个字节，剩下的50个字节不读，它也不会再次触发。因此在ET模式下，一定要把‘读缓冲区’的数据一次性读完。

在实际开发中，一般倾向于LT，这也是默认的模式。java NIO(New IO库)用的也是epoll的LT模式。因为ET容易漏事件，一次触发如果没有处理好，就没有第二次机会了。虽然LT重复触发可能有少许的性能消耗，但代码写起来更安全。

##### d) IO多路复用伪代码
```c
while(true) {
    select(...) 或者 poll(...) 或者 epoll_wait(...)
    if(某个fd有读事件) {
        read(fd,...)
    }
    if(某个fd有写事件) {
        write(fd,...)
    }
}
```

整个epoll的过程分为三个步骤：
* 事件注册，通过epoll_ctl实现。
对于服务器而言，是accept、read、write三种事件
对于客户端而言，是connect、read、write三种事件
* 轮询这三个事件是否就绪，通过epoll_wait实现，有事件发生，该函数返回。
* 事件就绪，执行实际的IO操作，通过accept/read/write实现

解释一下什么是‘事件就绪’
* read事件就绪：远程有新数据来了，socket读取缓冲区里的数据，需要调用read函数处理。
* write事件就绪：本地的socket写缓冲区是否可写，如果写缓冲区没有满，则一直是可写的，write事件一直是就绪的，可以调用write函数。只有当遇到发送大文件的场景，socket写缓冲区被占满时，write事件才不是就绪状态。
* accept事件就绪：有新的连接进入，需要调用accept函数处理。

### 4、异步IO----同步IO
异步就是异步，没有阻塞和非阻塞之分。

也叫异步非阻塞IO，经典的Proactor设计模式。

windows系统中的IOCP，这是一种真正意义上的异步IO。

所谓异步IO，是指读写都是由操作系统完成的，然后通过回调函数或者某种其他通信机子通知应用程序。

## 三、上层网络框架封装的网络IO模型
### 1、C++跨平台的网络库:asio,异步IO

AISO的‘异步’，是‘真异步’，还是‘假异步’？
* 在linux系统上封装的是epoll，不算真异步，只是IO多路复用
* 在windows系统上封装的是IOCP,是真异步

### 2、linux下java:NIO，Netty，epoll
java NIO和Linux epoll API对比：几乎一样

|  | java NIO| epoll |
| --- | --- |--- |
| 注册 |channel.register(selector,XXX) selectKey.interOps = XXX| epoll_ctr(...)|
|轮询| selector.poll()| epoll.wait(...) |
|实际IO操作| channel.accept channel.read channel.write| accept read write |

### 3、2个网络io的设计模式
操作系统的网络IO模型设计和上层网络框架的网络io模型的设计，用的都是这两种设计模式之一。
* Reactoer模式：主动模式，是指应用程序不断地轮询，询问操作系统或者网络框架、IO是否就绪。
    linux系统下的select/poll/epoll,java中的NIO都属于这种模式。在这种模式下，实际的IO操作还是应用程序执行的。

    好处：更加方便网络操作与业务进行分离。

* Proactor模式：被动模式，应用程序 把read和write函数操作全部交给操作系统或者网络框架，实际的IO操作由操作系统或网络框架完成，之后再回调应用程序。asio库就是典型的Proactor模型。

## 四、应用程序的网络IO于线程模型
### 1、普通网络IO模型
![20220226092142.jpg](https://pic.imgdb.cn/item/621980bf2ab3f51d91340817.jpg)

* 监听线程：负责accept事件的注册和处理。和每一个新进来的客户端建立socket连接，然后把socket连接移交给IO线程，完成任务，继续监听新的客户端。
* IO线程：负责每个socket连接上面read/write事件的注册和实际的socket的读写。把request放入request队列，交由worker线程处理。
* worker线程:纯碎的业务线程，没有socket读写操作。对request队列进行处理，生成response队列，然后写入response队列，由IO线程再回复给客户端。

##### a) 为什么叫做半同步半异步？
1-n是多路io复用 （epoll），异步 (包括请求接受，处理，解码，返回) 。worker是同步的（比如同步掉用数据库，等）。

##### b) 上面模型的N、M取值分别多大合适？
N：cpu的核数
M:M=cpu核数/(cpu时间/(cpu时间+IO时间))
比如cpu时间和IO时间为1:1，则线程为2*cpu核数

### 2、tomact 6的网络IO模型
![F33F6894-9C39-4A47-8BAA-B8990A0EE40B.png](https://pic.imgdb.cn/item/6219df2f2ab3f51d910737ec.png)
IO线程只负责read/write事件的注册和监听，执行了epoll里面的前两个阶段，第三个阶段是在worker线程里面做的。IO线程监听到一个socket连接上有读事件，于是把socket移交给worker线程，worker线程读出数据，处理完业务逻辑，直接返回给客户端。

* IO线程和worker线程之间交互，不再需要一来一回两个队列，直接是一个socket集合。
* worker线程不会被IO阻塞，是因为IO线程检测到读事件就绪之后，才把socket交给worker线程处理。

### 3、实际应用
实际应用中应用程序不需要自己开线程池，比如Netty框架内部已经把1，N，M 3个对应的线程池准备好了。