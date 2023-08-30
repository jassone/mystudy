## 什么是socket(Socket Api)
## 一、简介
Socket是BSD UNIX的进程通信机制，通常也称作”套接字”，用于描述IP地址和端口，是一个通信链的句柄。<font color="red">Socket可以理解为TCP/IP网络的API</font>，它定义了许多函数或例程，程序员可以用它们来开发TCP/IP网络上的应用程序。电脑上运行的应用程序通常通过”套接字”向网络发出请求或者应答网络请求。

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是`一组接口`。在设计模式中，Socket其实就是一个`门面模式`，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。而我们所说的socket编程指的是利用soket接口来实现自己的业务和协议。

Socket 连接,至少需要**一对**套接字。

## 二、socket编程
### 1、socket四元组(某人定义的叫法)
源IP+port+目标IP+port, 构成绝对唯一的连接

### 2、socket图解

![20190718154523875.png](https://pic.imgdb.cn/item/610fb1945132923bf8916cc3.png)

常用的Socket类型有两种：**流式Socket和数据报式Socket**

- **流式Socket是一种面向连接的Socket，针对于面向连接的TCP服务应用。**
- **数据报式Socket是一种无连接的Socket，针对于无连接的UDP服务应用**。

### 3、流程
如果一个程序创建了一个socket，并让其监听80端 口，其实是向TCP/IP协议栈声明了其对80端口的占有。以后，所有目标是80端口的TCP数据包都会转发给该程序（这里的程序，因为使用的是 Socket编程接口，所以首先由Socket层来处理）。所谓accept函数，其实抽象的是TCP的连接建立过程。accept函数返回的新 socket其实指代的是本次创建的连接，而一个连接是包括两部分信息的，一个是源IP和源端口，另一个是宿IP和宿端口。所以，accept可以产生多 个不同的socket，而这些socket里包含的宿IP和宿端口是不变的，变化的只是源IP和源端口。这样的话，这些socket宿端口就可以都是 80，而Socket层还是能根据源/宿对来准确地分辨出IP包和socket的归属关系，从而完成对TCP/IP协议的操作封装！

![c023b84_1440w.jpeg](https://pic.imgdb.cn/item/62dea40df54cd3f93791a075.jpg)

## 三、两种接收发送消息方式

todo

 https://blog.csdn.net/jia12216/article/details/82702960

## 四、其他
### 1、socket和TCP/IP，http的关系
* Socket是软件。TCP是协议。
* Socket 是应用层调用网络层的接口。
TCP 是网络层。

Socket是通信上一种通信方式， TCP是一种通信协议，我们所开发的程序大都都是基于TCP协议的socket通信，当然也可以使用其他协议来通过socket通信，比如 UDP， 更强悍的话，你可以自定义协议来通过socket进行网络通信。

**socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API）**，通过Socket，我们才能使用TCP/IP协议。 

Socket跟TCP/IP协议没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以说，Socket的出现 只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口，比如create、 listen、connect、accept、send、read和write等等。

Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接，Socket是发动机，提供了网络通信的能力。

实际上，传输层的TCP是基于网络层的IP协议的，而应用层的HTTP协议又是基于传输层的TCP协议的，而Socket本身不算是协议，它只是提供了一个针对TCP或者UDP编程的接口。

HTTP是轿车，提供了封装或者显示数据的具体形式。

> “TCP/IP只是一个协议栈，就像操作系统的运行机制一样，必须要具体实现，同时还要提供对外的操作接口。这个就像操作系统会提供标准的编程接口，比如win32编程接口一样，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口。” 

## 五、相关问题
### 1、关于socket长连接 CPU占用较高解决办法
在接受和发送的white循环里面停顿10毫秒。Thread.sleep(10);
todo


## 六、相关wiki
* todo 记一次Socket.IO长链服务的性能压测 https://zhuanlan.zhihu.com/p/45701773?ivk_sa=1024320u
