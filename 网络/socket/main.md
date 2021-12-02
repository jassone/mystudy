## 什么是socket(Socket Api)
###  socket编程
Socket是BSD UNIX的进程通信机制，通常也称作”套接字”，用于描述IP地址和端口，是一个通信链的句柄。<font color="red">Socket可以理解为TCP/IP网络的API</font>，它定义了许多函数或例程，程序员可以用它们来开发TCP/IP网络上的应用程序。电脑上运行的应用程序通常通过”套接字”向网络发出请求或者应答网络请求。

###  socket四元组(某人定义的叫法)
源IP+port+目标IP+port, 构成绝对唯一的连接

###  socket图解
* Socket是应用层与TCP/IP协议族通信的中间软件抽象层，<font color="red">它是一组接口。</font>
* 在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说只需要调用Socket规定的相关函数，让Socket去组织数据，以符合指定的协议。

![20190718154523875.png](https://pic.imgdb.cn/item/610fb1945132923bf8916cc3.png)

* Socket又称“套接字”，应用程序通常通过“套接字”向网络发出请求或者应答网络请求
* 常用的Socket类型有两种：流式Socket和数据报式Socket，流式是一种面向连接的Socket，针对于面向连接的TCP服务应用。数据报式Socket是一种无连接的Socket，针对于无连接的UDP服务应用。
 

