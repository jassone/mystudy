## IO模型演化

## 一、阻塞IO(BIO)-linux传统的IO模型

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

这肯定是不行的。浪费性能，可以使用非阻塞IO优化。

## 二、非阻塞 IO(NIO)

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

**不过，这不叫非阻塞 IO，只不过用了多线程的手段使得主线程没有卡在 read 函数上不往下走罢了。操作系统为我们提供的 read 函数仍然是阻塞的。**

所以真正的非阻塞 IO，不能是通过我们用户层的小把戏，而是**要恳请操作系统为我们提供一个非阻塞的 read 函数。这个 read 函数的效果是，如果没有数据到达时（到达网卡并拷贝到了内核缓冲区），立刻返回一个错误值（-1），而不是阻塞地等待。**

操作系统提供了这样的功能，只需要在调用 read 前，将文件描述符设置为非阻塞即可。

```c
fcntl(connfd, F_SETFL, O_NONBLOCK);
int n = read(connfd, buffer) != SUCCESS);
```

这样，就需要用户线程循环调用 read，直到返回值不为 -1，再开始处理业务。

![640tyuyytrew.gif](https://pic.imgdb.cn/item/621b38a62ab3f51d91a938a4.gif)

这里我们注意到一个细节。

- 非阻塞的 read，指的是在数据到达前，即数据还未到达网卡，或者到达网卡但还没有拷贝到内核缓冲区之前，这个阶段是非阻塞的。

- 当数据已到达内核缓冲区，此时调用 read 函数仍然是阻塞的，需要等待数据从内核缓冲区拷贝到用户缓冲区，才能返回。

大致经历两个阶段：

- 等待数据阶段：未阻塞， 用户进程需要盲等，**不停的去轮询内核**。
- 数据复制阶段：阻塞，此时进行数据复制。

整体流程如下图
![640qewerty.png](https://pic.imgdb.cn/item/621b39002ab3f51d91a9dd39.png)

缺点：

它相对于阻塞IO，虽然大幅提升了性能，但是它依然存在**性能问题**，即**频繁的轮询**，导致频繁的系统调用，同样会消耗大量的CPU资源。可以考虑**IO复用模型**，去解决这个问题。

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

既然**NIO**无效的轮询会导致CPU资源消耗，我们等到内核数据准备好了，主动通知应用进程再去进行系统调用，那不就好了嘛？

所以，还是得恳请操作系统老大，提供给我们一个有这样效果的函数，我们将一批文件描述符通过一次系统调用传给内核，由内核层去遍历，才能真正解决这个问题。

IO复用模型核心思路：系统给我们提供**一类函数**（如select、poll、epoll函数），它们可以同时监控多个fd的操作，任何一个返回内核数据就绪，应用进程再发起recvfrom系统调用。

### 1、select

应用进程通过调用**select**函数，可以同时监控多个fd(事先加入的)，在select函数监控的fd中，只要有任何一个数据状态准备就绪了，select函数就会返回可读状态，这时应用进程再发起recvfrom请求去读取数据。

![640.gif](https://pic.imgdb.cn/item/621b3a282ab3f51d91abefc4.gif)

select系统调用的函数定义如下。
```c
int select(
    int nfds, // nfds:监控的文件描述符集里最大文件描述符加1
    fd_set *readfds, // readfds：监控有读数据到达文件描述符集合，传入传出参数
    fd_set *writefds, // writefds：监控写数据到达文件描述符集合，传入传出参数
    fd_set *exceptfds, // exceptfds：监控异常发生达文件描述符集合, 传入传出参数
    struct timeval *timeout 
    // timeout：定时阻塞监控时间，3种情况
    //  1.NULL，永远等下去
    //  2.设置timeval，等待固定时间
    //  3.设置timeval里时间均为0，检查描述字后立即返回，轮询
);
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

不过，当 select 函数返回后，用户依然需要遍历刚刚提交给操作系统的 list。只不过，操作系统会将准备就绪的文件描述符做上标识，用户层将不会再有无意义的系统调用开销。

可以看出几个细节：
1. select 每次调用需要传入 fd 数组，都需要拷贝一份到内核，影响效率。高并发场景下这样的拷贝消耗的资源是惊人的。（可优化为不复制）
2. select 在内核层仍然是通过遍历的方式检查文件描述符的就绪状态，是个同步过程，只不过无系统调用切换上下文的开销。（内核层可优化为异步事件通知）
3. select 仅仅返回可读文件描述符的个数，具体哪个可读还是要用户自己遍历。（可优化为只返回给用户就绪的文件描述符，无需用户做无效的遍历）

整个 select 的流程图如下。
![640.png](https://pic.imgdb.cn/item/621b3b4a2ab3f51d91aded81.png)

可以看到，这种方式，既做到了一个线程处理多个客户端连接（文件描述符），又减少了系统调用的开销（多个文件描述符只有一次 select 的系统调用 + n 次就绪状态的文件描述符的 read 系统调用）。

优点：

非阻塞IO模型（NIO）中，需要N（N>=1）次轮询系统调用，然而借助select的IO多路复用模型，**只需要发起一次询问**就够了,大大优化了性能。

缺点：

- 监听的IO最大连接数有限，在Linux系统上一般为1024。因为**存在连接数限制**，所以后来又提出了**poll**。与select相比，**poll**解决了**连接数限制问题**。
- select函数返回后，是通过**遍历**fdset，找到就绪的描述符fd。（仅知道有I/O事件发生，却不知是哪几个流，所以**遍历所有流**）。

但是呢，select和poll一样，还是需要通过遍历文件描述符来获取已经就绪的socket。如果同时连接的大量客户端，在一时刻可能只有极少处于就绪状态，伴随着监视的描述符数量的增长，**效率也会线性下降**。

因此经典的多路复用模型epoll诞生。

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
epoll 是最终的大 boss，采用事件驱动模式。

epoll 主要就是针对select部分的三个细节问题进行了改进。
1. 内核中保存一份文件描述符集合，无需用户每次都重新传入，只需告诉内核修改的部分即可。
2. 内核不再通过轮询的方式找到就绪的文件描述符，而是通过**异步 IO 事件唤醒**。
3. 内核仅会将有 IO 事件的文件描述符返回给用户，用户也无需遍历整个文件描述符集合。

原理：

**epoll**先通过epoll_ctl()来注册一个fd（文件描述符），一旦基于某个fd就绪时，内核会采用回调机制，迅速激活这个fd，当进程调用epoll_wait()时便得到通知。这里去掉了**遍历文件描述符**的坑爹操作，而是采用**监听事件回调**的机制。这就是epoll的亮点。

操作系统提供了这三个函数。

- 第一步，创建一个 epoll 句柄，epoll_create()。
- 第二步，向内核添加、修改或删除要监控的文件描述符， epoll_ctl()。
- 第三步，该函数用于轮询I/O事件的发生,类似发起了select() 调用，epoll_wait()。

使用起来，其内部原理就像如下一般丝滑。
![wertytrewrtr.gif](https://pic.imgdb.cn/item/621b3c462ab3f51d91afa1d4.gif)

##### Reactor模式

Linux下的select、poll和epoll将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。

这样，**整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。**

##### epoll的缺点

epoll明显优化了IO的执行效率，但在进程调用epoll_wait()时，仍然可能被阻塞。

能不能酱紫：不用我老是去问你数据是否准备就绪，等我发出请求后，你数据准备好了通知我就行了，这就诞生了**信号驱动IO模型**。

## 四、信号驱动IO模型

信号驱动IO不再用主动询问的方式去确认数据是否就绪，而是向内核发送一个信号（调用sigaction的时候建立一个SIGIO的信号），然后应用用户进程可以去做别的事，不用阻塞。当内核数据准备好后，再通过SIGIO信号通知应用进程，数据准备好后的可读状态。应用用户进程收到信号之后，立即调用recvfrom，去读取数据。

![71cf3bc79f3df8dc46e8fa82451028ea.png](https://pic.imgdb.cn/item/62dd1892f54cd3f9379978f0.png)

信号驱动IO模型，在应用进程发出信号后，是立即返回的，不会阻塞进程。它已经有异步操作的感觉了。

但是数据复制到应用缓冲的时候**，应用进程还是阻塞的。回过头来看下，不管是BIO，还是NIO，还是信号驱动，在数据从内核复制到应用缓冲的时候，都是阻塞的。**

还有没有优化方案呢？AIO（真正的异步IO）。

## 五、异步IO(AIO)

前面讲的BIO，NIO和信号驱动，在数据从内核复制到应用缓冲的时候，都是**阻塞**的，因此都不算是真正的异步。AIO实现了IO全流程的非阻塞，就是应用进程发出系统调用后，是立即返回的，但是**立即返回的不是处理结果，而是表示提交成功类似的意思**。等内核数据准备好，将数据拷贝到用户进程缓冲区，发送信号通知用户进程IO操作执行完毕。

![d16a887ba0d2408b1.png](https://pic.imgdb.cn/item/62dd1acff54cd3f937a73614.png)

异步IO的优化思路很简单，只需要向内核发送一次请求，就可以完成数据状态询问和数据拷贝的所有操作，并且不用阻塞等待结果。

日常开发中，有类似思想的业务场景：比如发起一笔批量转账，但是批量转账处理比较耗时，这时候后端可以先告知前端转账提交成功，等到结果处理完，再通知前端结果即可。

缺点：

- 异步I/O在Linux2.6才引入，而且到现在仍然未成熟。虽然有知名的异步I/O库 glibc，但是glibc采用多线程模拟，但存在一些bug和设计上的不合理。
- 引入异步I/O可能会代码难以理解的问题，这个站在软件工程的角度也是需要细细衡量的。
  

## 六、大白话解释什么是I/O多路复用

### 1、场景

假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：
* 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡主，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。 
* 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 
* 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。 

### 2、场景分析
* 第一种就是阻塞IO模型。
* 第二种类似于为每一个用户创建一个进程或者线程处理连接。 
* 第三种就是I/O复用模型。

## 七、总结

一切的开始，都起源于这个 **read 函数是操作系统提供的，而且是阻塞的，我们叫它 阻塞 IO。**

为了破这个局，程序员在用户态通过多线程来防止主线程卡死。
后来操作系统发现这个需求比较大，于是在操作系统层面提供了非阻塞的 read 函数，这样程序员就可以在一个线程内完成多个文件描述符的读取，这就是 非阻塞 IO。

但多个文件描述符的读取就需要遍历，当高并发场景越来越多时，用户态遍历的文件描述符也越来越多，相当于在 while 循环里进行了越来越多的系统调用。

后来操作系统又发现这个场景需求量较大，于是又在操作系统层面提供了这样的遍历文件描述符的机制，这就是 IO 多路复用。
多路复用有三个函数，最开始是 select，然后又发明了 poll 解决了 select 文件描述符的限制，然后又发明了 epoll 解决 select 的三个不足。

所以，IO 模型的演进，其实就是时代的变化，倒逼着操作系统将更多的功能加到自己的内核而已。

## 八、参考wiki

* 你管这破玩意叫 IO 多路复用？ https://mp.weixin.qq.com/s/YdIdoZ_yusVWza1PU7lWaw

* 图解 | 深入揭秘 epoll 是如何实现 IO 多路复用的！ https://zhuanlan.zhihu.com/p/361750240

* 看一遍就理解：IO模型详解 https://baijiahao.baidu.com/s?id=1718409483059542510&wfr=spider&for=pc

