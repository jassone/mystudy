## TCP半连接和全连接

当服务端调用listen函数监听端口的时候，内核会为每个监听的socket创建两个队列：

- **半连接队列(syn queue**)：客户端发送`SYN`包，服务端收到后回复`SYN+ACK`后，服务端进入`SYN_RCVD`状态，这个时候的socket会放到半连接队列。
- **全连接队列(accept queue)**：当服务端收到客户端的ACK后，socket会从半连接队列移出到全连接队列。当调用accpet函数的时候，会从全连接队列的头部返回可用socket给用户进程。



todo
https://zhuanlan.zhihu.com/p/99152064?from_voters_page=true