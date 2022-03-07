## reactor

异步非阻塞模型 Reactor


## 特性
Reactor有4个核心的操作：
* add 添加socket监听到reactor
* set 修改事件监听，可以设置监听的类型，如可读、可写
* del 从reactor中移除，不再监听事件
* callback 就是事件发生后对应的处理逻辑，一般在add/set时制定。

## 应用
Reactor模型还可以与多进程、多线程结合起来用，既实现异步非阻塞IO，又利用到多核。
目前流行的异步服务器程序都是这样的方式：如
* Nginx：多进程Reactor
* Nginx+Lua：多进程Reactor+协程
* Golang：单线程Reactor+多线程协程
* Swoole：多线程Reactor+多进程Worker 
 