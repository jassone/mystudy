## redis通信协议-RESP
Redis客户端和服务端之间使用一种名为RESP(REdis Serialization Protocol)的**二进制安全文本协议**进行通信。RESP在位于TCP之上，而网络模型上客户端和服务器是保持的**双工**的连接。虽然该协议是<font color="red">专为Redis设计的</font>，但它可以用于其他客户端-服务器软件项目。

### 高性能 Redis 协议分析器
尽管 Redis 的协议非常利于人类阅读， 定义也很简单， **但这个协议的实现性能仍然可以和二进制协议一样快**。

**因为 Redis 协议将数据的长度放在数据正文之前， 所以程序无须像 JSON 那样， 为了寻找某个特殊字符而扫描整个 payload ， 也无须对发送至服务器的 payload 进行转义。**

### RESP应用场景
1. 开发定制化的客户端. RESP设计成简单的文本协议, 一大原因就是为了降低各种语言开发客户端的复杂度
2. 理解RESP方便我们分析AOF文件,了解redis的内部设计
3. 平时通过抓包软件,可以帮助快速定位redis的相关问题
4. 在没有redis-cli的情况下, 方便开发调试redis命令

### RESP数据类型
一般来说,RESP只需要序列化三种数组即可: 字符串, 整数, 数组. 而在实际场景中, RESP又把字符串细化成了simple string, error string和bulk string三种.

1.simple string. 简单的字符串
2.error. 就是表示这是一个错误(异常)情况
3.integer 表示这是一个整数
4.bulk string. 表示是长字符串,但是必须小于512M.
5.arrays. 表示这是一个数组,数组元素可以是上面的任意一种类型,也可以是一个数组

### 通信过程 - request-response模型

一般来说,redis客户端和服务端交互都是通过以下两个步骤:

 1. redis发送一个命令到服务端, 然后阻塞在socket.read()方法, 等待服务端的返回
 2. 服务端收到一个命令, 处理完成后将数据发送回去给客户端


这个就被称为request/reponse模型. redis的大部分命令都是使用这种模型进行通讯, 除了两种情况:

  1. pipeline模式. 在pipeline模式下, 客户端可能会把多个命令收集在一起, 然后一并发送给服务端, 最后等待服务端把所有命令的执行响应一并发送回来
  2. pub/sub, 发布订阅模式下, redis客户端只需要发送一次订阅命令

RESP协议的request/response模型可以总结为以下两个步骤

 1. 客户端发送命令, 一般组装成bulk string的数组
 2. 服务端处理命令, 根据不同的命令,可能返回不同的数据类型

> client->resp序列化->request->server->resp序列化->response->client

Redis的通信协议首先是以行来划分，每行以\r\n行结束。每一行都有一个消息头，请求消息头共分为2种分别如下:
* (\*）表示消息体总共有多少行，不包括当前行,*后面是具体的行数。
* ($)表示下一行数据长度，不包括换行符长度\r\n,后面则是对应的长度的数据。

举个例子：
```
*3\r\n  #消息一共有三行
$3\r\n #第一行有长度为3
set\r\n #第一行的消息
$4\r\n  #第二行长度为4 
demo\r\n #第二行的消息
$6\r\n #第三行长度为6
123456\r\n #第三行的消息
+OK\r\n #操作成功
```

##### response
* 对于简单字符串，答复的第一个字节为“ +”
* 对于错误，回复的第一个字节为“-”
* 对于整数，答复的第一个字节为“:”
* 对于指批量、多行字符串(<=512M)，答复的第一个字节为“ $”，后跟组成字符串的<font color="red">字节数（带前缀的长度)</font>
* 对于数组，回复的第一个字节为“ *”

```
demo

+OK\r\n 成功
-ERR unknown command 'foobar'\r\n 失败
：0\r\n  返回整数
$6\r\nfoobar\r\n 字符串
$0\r\n\r\n 空字符串
$-1\r\n  Null， 在这种特殊格式中，长度为-1，并且没有数据
*0\r\n 空数组
*-1\r\n null数组，比如当BLPOP命令超时时
*2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n  散列字符串“ foo”和“ bar”的数组
*3\r\n:1\r\n:2\r\n:3\r\n 三个整数的数组

给key增加过期时间(原始命令 expire name 10)
*3
$9
PEXPIREAT
$4
name
$13
1627717194639
```

一张图说明
 ![1](https://pic.imgdb.cn/item/6104d2b25132923bf8584510.png)

##### telnet方式使用

```
my-Mac:~ lhh$ telnet localhost 6379
 
 // redis 命令模式
set test1 1
+OK

get test1
$1
1

//RESP模式，也可以发送RESP格式的命令, 但是要在本文编辑器里面把\r\n换成换行符, 再复制过去,不然会报错
*2
$3
get
$5
test1
$1
1
```

##### socket方式使用
redis是基于tcp通讯的, 所以简单使用socket就好, 发送如下字符串 
```
"*2\r\n$3\r\nget\r\n$5\r\ntest1\r\n"
```

