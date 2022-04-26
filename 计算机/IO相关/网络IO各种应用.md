##
![20220227103619.jpg](https://pic.imgdb.cn/item/621ae3da2ab3f51d91ec811e.jpg)


todo***

异步IO： 比如AIO 
IO异步操作：

异步：多个线程会公用fd，多个线程同时对fd读与写(造成会有脏数据), 解决：只能加锁

accept() 几千/s没问题

多个线程accept去抢，惊群问题：解决：加锁，谁占有了锁就抢到了


假如16核处理器，4个处理accept，12个处理recv/send， memcached就是这样的模型(但是只有一个accept)
优点：redv/send 不会影响accept，偏向于适用于有业务需求项目，即时通讯等

但是redis为什么比mencached快？
redis是纯内存操作,没有太多的IO操作，耗时的只有recv
memcached中有加锁操作，是一种负担

而如果类似mysql这种需要读取磁盘的操作，单线程肯定是不行的，因为并发上不去


多进程-典型例子nginx，为什么？
因为是web服务器，每一个请求时独立，session是独立的



##### IO
BIO
NIO
epoll
