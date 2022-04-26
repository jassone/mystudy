## IO 多路复用演化
## 一、阻塞IO-linux传统的IO模型
服务端为了处理客户端的连接和请求的数据，写了如下伪代码。

```c
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  int n = read(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```
这段代码会执行得磕磕绊绊，就像这样。
![eretrerer.gif](https://pic.imgdb.cn/item/621b36722ab3f51d91a3e4dc.gif)

可以看到，服务端的线程阻塞在了两个地方，一个是 accept 函数，一个是 read 函数。

如果再把 read 函数的细节展开，我们会发现其阻塞在了两个阶段。

![640fyuytrerthj.gif](https://pic.imgdb.cn/item/621b36aa2ab3f51d91a46da8.gif)

这就是传统的阻塞 IO。

整体流程如下图。
![dfefwefwefew.png](https://pic.imgdb.cn/item/621b36e82ab3f51d91a509dd.png)

所以，如果这个连接的客户端一直不发数据，那么服务端线程将会一直阻塞在 read 函数上不返回，也无法接受其他客户端连接。

这肯定是不行的。

## 二、非阻塞 IO
为了解决上面的问题，其关键在于改造这个 read 函数。

有一种聪明的办法是，每次都创建一个新的进程或线程，去调用 read 函数，并做业务处理。

```c
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  pthread_create（doWork);  // 创建一个新的线程
}
void doWork() {
  int n = read(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```

这样，当给一个客户端建立好连接后，就可以立刻等待新的客户端连接，而不用阻塞在原客户端的 read 请求上。

![640dfghgr.gif](https://pic.imgdb.cn/item/621b384a2ab3f51d91a88e3c.gif)

不过，这不叫非阻塞 IO，只不过用了多线程的手段使得主线程没有卡在 read 函数上不往下走罢了。操作系统为我们提供的 read 函数仍然是阻塞的。

所以真正的非阻塞 IO，不能是通过我们用户层的小把戏，而是要恳请操作系统为我们提供一个非阻塞的 read 函数。

这个 read 函数的效果是，如果没有数据到达时（到达网卡并拷贝到了内核缓冲区），立刻返回一个错误值（-1），而不是阻塞地等待。

操作系统提供了这样的功能，只需要在调用 read 前，将文件描述符设置为非阻塞即可。

```c
fcntl(connfd, F_SETFL, O_NONBLOCK);
int n = read(connfd, buffer) != SUCCESS);
```

这样，就需要用户线程循环调用 read，直到返回值不为 -1，再开始处理业务。

![640tyuyytrew.gif](https://pic.imgdb.cn/item/621b38a62ab3f51d91a938a4.gif)

这里我们注意到一个细节。

非阻塞的 read，指的是在数据到达前，即数据还未到达网卡，或者到达网卡但还没有拷贝到内核缓冲区之前，这个阶段是非阻塞的。

当数据已到达内核缓冲区，此时调用 read 函数仍然是阻塞的，需要等待数据从内核缓冲区拷贝到用户缓冲区，才能返回。

整体流程如下图
![640qewerty.png](https://pic.imgdb.cn/item/621b39002ab3f51d91a9dd39.png)

## 三、IO 多路复用
为每个客户端创建一个线程，服务器端的线程资源很容易被耗光。

![640tyuytrew.png](https://pic.imgdb.cn/item/621b396d2ab3f51d91aa9c22.png)

当然还有个聪明的办法，我们可以每 accept 一个客户端连接后，将这个文件描述符（connfd）放到一个数组里。

```c
fdlist.add(connfd);
```

然后弄一个新的线程去不断遍历这个数组，调用每一个元素的非阻塞 read 方法。
```c
while(1) {
  for(fd <-- fdlist) {
    if(read(fd) != -1) {
      doSomeThing();
    }
  }
}
```

这样，我们就成功用一个线程处理了多个客户端连接。
![640iuytrewergf.gif](https://pic.imgdb.cn/item/621b2f122ab3f51d91910442.gif)

你是不是觉得这有些多路复用的意思？

但这和我们用多线程去将阻塞 IO 改造成看起来是非阻塞 IO 一样，这种遍历方式也只是我们用户自己想出的小把戏，每次遍历遇到 read 返回 -1 时仍然是一次浪费资源的系统调用。

在 while 循环里做系统调用，就好比你做分布式项目时在 while 里做 rpc 请求一样，是不划算的。

所以，还是得恳请操作系统老大，提供给我们一个有这样效果的函数，我们将一批文件描述符通过一次系统调用传给内核，由内核层去遍历，才能真正解决这个问题。

### 1、select
select 是操作系统提供的系统调用函数，通过它，我们可以把一个文件描述符的数组发给操作系统， 让操作系统去遍历，确定哪个文件描述符可以读写， 然后告诉我们去处理：

![640.gif](https://pic.imgdb.cn/item/621b3a282ab3f51d91abefc4.gif)

select系统调用的函数定义如下。
```c
int select(
    int nfds,
    fd_set *readfds,
    fd_set *writefds,
    fd_set *exceptfds,
    struct timeval *timeout);
// nfds:监控的文件描述符集里最大文件描述符加1
// readfds：监控有读数据到达文件描述符集合，传入传出参数
// writefds：监控写数据到达文件描述符集合，传入传出参数
// exceptfds：监控异常发生达文件描述符集合, 传入传出参数
// timeout：定时阻塞监控时间，3种情况
//  1.NULL，永远等下去
//  2.设置timeval，等待固定时间
//  3.设置timeval里时间均为0，检查描述字后立即返回，轮询
```
服务端代码，这样来写。

首先一个线程不断接受客户端连接，并把 socket 文件描述符放到一个 list 里。

```c
while(1) {
  connfd = accept(listenfd);
  fcntl(connfd, F_SETFL, O_NONBLOCK);
  fdlist.add(connfd); // 把fd扔到list数组里
}
```

然后，另一个线程不再自己遍历，而是调用 select，将这批文件描述符 list 交给操作系统去遍历。
```c
while(1) {
  // 把一堆文件描述符 list 传给 select 函数
  // 有已就绪的文件描述符就返回，nready 表示有多少个就绪的
  nready = select(list);
  ...
}
```

不过，当 select 函数返回后，用户依然需要遍历刚刚提交给操作系统的 list。

只不过，操作系统会将准备就绪的文件描述符做上标识，用户层将不会再有无意义的系统调用开销。
```c
while(1) {
  nready = select(list);
  // 用户层依然要遍历，只不过少了很多无效的系统调用
  for(fd <-- fdlist) {
    if(fd != -1) {
      // 只读已就绪的文件描述符
      read(fd, buf);
      // 总共只有 nready 个已就绪描述符，不用过多遍历
      if(--nready == 0) break;
    }
  }
}
```

可以看出几个细节：
1. select 每次调用需要传入 fd 数组，都需要拷贝一份到内核，影响效率。高并发场景下这样的拷贝消耗的资源是惊人的。（可优化为不复制）
2. select 在内核层仍然是通过遍历的方式检查文件描述符的就绪状态，是个同步过程，只不过无系统调用切换上下文的开销。（内核层可优化为异步事件通知）
3. select 仅仅返回可读文件描述符的个数，具体哪个可读还是要用户自己遍历。（可优化为只返回给用户就绪的文件描述符，无需用户做无效的遍历）

整个 select 的流程图如下。
![640.png](https://pic.imgdb.cn/item/621b3b4a2ab3f51d91aded81.png)

可以看到，这种方式，既做到了一个线程处理多个客户端连接（文件描述符），又减少了系统调用的开销（多个文件描述符只有一次 select 的系统调用 + n 次就绪状态的文件描述符的 read 系统调用）。

### 2、poll
poll 也是操作系统提供的系统调用函数。

int poll(struct pollfd *fds, nfds_tnfds, int timeout);

```c
struct pollfd {
  intfd; /*文件描述符*/
  shortevents; /*监控的事件*/
  shortrevents; /*监控事件中满足条件返回的事件*/
};
```
它和 select 的主要区别就是，去掉了 select 只能监听 1024 个文件描述符的限制。

### 3、epoll
epoll 是最终的大 boss，它解决了 select 和 poll 的一些问题。

epoll 主要就是针对select部分的三个细节问题进行了改进。
1. 内核中保存一份文件描述符集合，无需用户每次都重新传入，只需告诉内核修改的部分即可。
2. 内核不再通过轮询的方式找到就绪的文件描述符，而是通过异步 IO 事件唤醒。
3. 内核仅会将有 IO 事件的文件描述符返回给用户，用户也无需遍历整个文件描述符集合。

具体，操作系统提供了这三个函数。

第一步，创建一个 epoll 句柄
```c
// size只是告诉内核，计划监听多少个fd。实际通过epoll_ctl添加的fd数目可能大于这个值。
int epoll_create(int size);
```

第二步，向内核添加、修改或删除要监控的文件描述符。
```c
int epoll_ctl(
  int epfd, int op, int fd, struct epoll_event *event);
```

第三步，该函数用于轮询I/O事件的发生,类似发起了 select() 调用
```c
//多长时间去轮询一次， timeout=-1表示如果IO没有数据，一直阻塞。 正数表示超时时间
//返回几个有数据可读或可写的IO
int epoll_wait(
  int epfd, struct epoll_event *events, int max events, int timeout);
```

使用起来，其内部原理就像如下一般丝滑。
![wertytrewrtr.gif](https://pic.imgdb.cn/item/621b3c462ab3f51d91afa1d4.gif)

##### a) 整个epoll的过程分为三个步骤：
* 事件注册，通过epoll_ctl实现。
对于服务器而言，是accept、read、write三种事件
对于客户端而言，是connect、read、write三种事件
* 轮询这三个事件是否就绪，通过epoll_wait实现，有事件发生，该函数返回。
* 事件就绪，执行实际的IO操作，通过accept/read/write实现

##### b) 解释一下什么是‘事件就绪’
* read事件就绪：远程有新数据来了，socket读取缓冲区里的数据，需要调用read函数处理。
* write事件就绪：本地的socket写缓冲区是否可写，如果写缓冲区没有满，则一直是可写的，write事件一直是就绪的，可以调用write函数。只有当遇到发送大文件的场景，socket写缓冲区被占满时，write事件才不是就绪状态。
* accept事件就绪：有新的连接进入，需要调用accept函数处理。

### 4、Reactor模式
Linux下的select、poll和epoll将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。

这样，**整个过程只在调用select、poll、epoll这些调用的时候才会阻塞**，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。

## 四、相关知识点
### 1、epoll的LT和ET模式
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

### 2、IO多路复用伪代码
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

## 五、大白话解释什么是I/O多路复用
### 1、场景
假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：
* 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡主，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。 
* 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 
* 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。 

### 2、场景分析
* 第一种就是阻塞IO模型。
* 第二种类似于为每一个用户创建一个进程或者线程处理连接。 
* 第三种就是I/O复用模型。

## 六、总结
大白话总结一下。
一切的开始，都起源于这个 read 函数是操作系统提供的，而且是阻塞的，我们叫它 阻塞 IO。

为了破这个局，程序员在用户态通过多线程来防止主线程卡死。
后来操作系统发现这个需求比较大，于是在操作系统层面提供了非阻塞的 read 函数，这样程序员就可以在一个线程内完成多个文件描述符的读取，这就是 非阻塞 IO。

但多个文件描述符的读取就需要遍历，当高并发场景越来越多时，用户态遍历的文件描述符也越来越多，相当于在 while 循环里进行了越来越多的系统调用。

后来操作系统又发现这个场景需求量较大，于是又在操作系统层面提供了这样的遍历文件描述符的机制，这就是 IO 多路复用。
多路复用有三个函数，最开始是 select，然后又发明了 poll 解决了 select 文件描述符的限制，然后又发明了 epoll 解决 select 的三个不足。

所以，IO 模型的演进，其实就是时代的变化，倒逼着操作系统将更多的功能加到自己的内核而已。

## 六、参考wiki
* 你管这破玩意叫 IO 多路复用？ https://mp.weixin.qq.com/s/YdIdoZ_yusVWza1PU7lWaw

* 图解 | 深入揭秘 epoll 是如何实现 IO 多路复用的！ https://zhuanlan.zhihu.com/p/361750240

