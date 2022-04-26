## canal
### 1、工作流程
* canal其实模拟的Mysql Slave的交互协议，**把自己伪装成Mysql Slave**，

    向Mysql Master发送一个dump请求；

* Master收到这个dump请求后，开始推送binlog日志给slave（这个slave指canal）；

* canal解析master发送过的binlog对象（当前binlog原始数据为byte流）

### 2、使用消息中间件来解耦
![3333333.png](https://pic.imgdb.cn/item/61d9747e2ab3f51d911131a7.png)

### 3、相关wiki
* 使用Canal实现MySQL的数据实时同步 https://blog.csdn.net/qq_23329167/article/details/89873870

* 基于canal实现mysql的数据同步 https://www.jianshu.com/p/50d1d8f06b75