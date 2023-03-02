## 网络IO各种应用

## 一、汇总

- Redis：异步IO+单线程

![20220227103619.jpg](https://pic.imgdb.cn/item/621ae3da2ab3f51d91ec811e.jpg)

## 二、二种高性能IO设计应用

### 1、Reactor （反应堆 同步IO）

- java NIO　　
- 多路复用IO　　
- redis 
- libevent 
- Linux
-  select/epoll
- kqueue

### 2、Proactor（前摄器 异步IO）

- java AIO　　
- 异步IO模型　　
- 目前只有 Windows IO（IOCP）

## 三、一些疑问

- 一般accept() 几千/s没问题
- 多个线程accept去抢，惊群问题：解决：加锁，谁占有了锁就抢到了
- 假如16核处理器，4个处理accept，12个处理recv/send， memcached就是这样的模型(但是只有一个accept)
  优点：redv/send 不会影响accept，偏向于适用于有业务需求项目，即时通讯等。
- 而如果类似mysql这种需要读取磁盘的操作，单线程肯定是不行的，因为并发上不去。
- 多进程-典型例子nginx，为什么？
  因为是web服务器，每一个请求时独立，session是独立的
