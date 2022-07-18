## RPC
## 一、PRC是什么
RPC (Remote Procedure Call)即远程过程调用，是分布式系统常见的一种通信方法。除RPC之外，常见的多系统数据交互方案还有分布式消息队列、HTTP请求调用、数据库和分布式缓存等。

**RPC是一种概念，HTTP也是RPC实现的一种方式**。

**通过RPC能解耦服务，这才是使用RPC的真正目的。RPC的原理主要用到了动态代理模式，是一个完整的远程调用方案，**至于HTTP协议，只是一个传输协议而已。

**主要是解决分布式系统中服务与服务直接的调用问题。**

## 二、主流产品

[phprpc](http://www.phprpc.org/zh_CN/)，[yar](https://github.com/laruence/yar), [thrift](http://thrift.apache.org/), [gRPC](http://www.grpc.io/), [swoole](http://www.swoole.com/), [hprose](https://github.com/hprose/hprose-php)

## 三、相关wiki

* https://zhuanlan.zhihu.com/p/280122318