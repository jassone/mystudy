## netcat
Netcat是一款非常出名的网络工具，简称“NC”,有渗透测试中的“瑞士军刀”之称。它可以用作端口监听、端口扫描、远程文件传输、还可以实现远程shell等功能。

Nc可以在两台设备上面相互交互，即侦听模式/传输模式。

## 一、参　　数：
* -d: 脱离命令窗口，在后台运行，常用于后门建立过程
* -e: 执行某个程序，常用于后门建立过程
* -G gateway: 设置网关，常用于突破内网限制
* -g num： 路由跳数
* -i sec:设置发送每一行数据的时间间隔
* -l: 设置netcat处于监听状态等待连接，**意味着nc被当作server，侦听并接受连**接。
* -L: 即使连接关闭，netcat依然处于监听状态，等待连接。
* -n: 设置netcat只识别ip地址，不在进行DNS解析。
* -o file:设置传输十六进制的数据
* -p port:设置本地监听的端口号
* -r: 设置netcat随机化的端口号
* -s addr:设置netcat源地址
* -t: 回复telnet的请求数据包
* -u: 设置netcat使用UDP模式，默认tcp模式
* -v: 显示交互或错误提示信息
* -w secs:设置连接超时秒数
* -z:表示使用Zero-I/O模式，即连接的时候禁止输入输出，仅查看端口是否开启。

## 二、使用nc发送get和post请求
##### GET

get请求头文件header.txt
```sh
GET / HTTP/1.0
Host: baidu.com
```

```sh
nc baidu.com 80 < header.txt
```

或者nc**_连接服务器_**后，再输入内容发送请求

```sh
➜  / nc www.baidu.com 80
GET / HTTP/1.1
(回车)
(回车)

HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 14615
 ...
```

或者多行一起
```sh
echo -e "GET / HTTP/1.0\r\nHost: www.baidu.com\r\n\r\n" | nc www.baidu.com 80
```
##### POST
post请求头文件header.txt，参数site=zixuephp.net test=test

```sh
POST / HTTP/1.0
Host: baidu.com
Content-Type: application/x-www-form-urlencoded;charset=utf-8
Content-Length: 27
 
site=zixuephp.net&test=test
```

```sh
nc baidu.com 80 < header.txt
```

## 三、其他用法
### 1、连接到远程主机后再进行通信(即与服务器进行交互，模拟客户端)
```sh
nc  -nvv Targert_IP  Targert_Port

➜  ~ git:(master) ✗ nc  -nvv 127.0.0.1 9501
Connection to 127.0.0.1 port 9501 [tcp/*] succeeded!

```

然后后面就可以发送对应的内容来和服务器进行交互，比如http请求(上面已介绍)，redis请求等。

```sh
//比如发送redis请求(keys *)
➜  / nc localhost 6379
keys *
*3
$2
...
```

udp连接
```sh
nc -u 127.0.0.1 9502

```

### 2、端口扫描
* 扫描指定主机的某个端口段的端口开放信息
```sh
~ git:(master) ✗ nc -zv 127.0.0.1  9501-9502
Connection to 127.0.0.1 port 9501 [tcp/*] succeeded!
nc: connectx to 127.0.0.1 port 9502 (tcp) failed: Connection refused
```

* 扫描指定主机的某个UDP端口段，并且返回端口信息
```sh
nc -zvu 127.0.0.1  9501-9502
```

### 3、端口监听
todo

### 4、正向 shell
todo

## 四、用例
### 1、简单聊天室
a) 开启服务端
```sh
$ nc -l 9999
```

b) 客户端连接服务端
```sh
$ nc 172.16.0.4 9999
```

c) 执行成功，两台服务器就建立了TCP连接，接着就能通过该连接发送消息了
```sh
$ nc 172.16.0.4 9999
Hello, I'm client
```

d) 服务端就能马上收到该消息
```sh
$ nc -l 9999
Hello, I'm client
```

同样的，在服务端发送的消息，客户端也能收到。

## 五、相关wiki
* https://segmentfault.com/a/1190000016626298