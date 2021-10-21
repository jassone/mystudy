**Netcat是一款非常出名的网络工具，简称“NC”,有渗透测试中的“瑞士军刀”之称。它可以用作端口监听、端口扫描、远程文件传输、还可以实现远程shell等功能**

## 参　　数：
* -d: 脱离命令窗口，在后台运行，常用于后门建立过程
* -e: 执行某个程序，常用于后门建立过程
* -G gateway: 设置网关，常用于突破内网限制
* -g num： 路由跳数
* -i sec:设置发送每一行数据的时间间隔
* -l: 设置netcat处于监听状态等待连接
* -L: 设置netcat处于监听状态，等待连接，当客户端断开，服务端依旧回到等待状态。
* -n: 设置netcat只识别ip地址，不在进行DNS解析。
* -o file:设置传输十六进制的数据
* -p port:设置本地监听的端口号
* -r: 设置netcat随机化的端口号
* -s addr:设置netcat源地址
* -t: 回复telnet的请求数据包
* -u: 设置netcat使用UDP模式
* -v: 显示错误提示信息
* -w secs:设置连接超时秒数
* -z:设置扫描模式，表示发送的数据包中不包含任何payload


### 使用nc发送get和post请求
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

### 连接到远程主机后再进行通信
```
nc  -nvv Targert_IP  Targert_Port

➜  ~ git:(master) ✗ nc  -nvv 127.0.0.1 9501
Connection to 127.0.0.1 port 9501 [tcp/*] succeeded!
```

然后后面就可以发送对应的内容来和服务器进行交互，比如http请求(上面已介绍)，redis请求等

```
//比如发送redis请求(keys *)
➜  / nc localhost 6379
keys *
*3
$2
...
```

### 监听本地主机 ???
```
nc  -l  -p  Local_Port
```

### 端口扫描
* 扫描指定主机的单一端口是否开放

```
nc  -v  target_IP  target_Port

➜  ~ git:(master) ✗ nc  -v  127.0.0.1 80          
Connection to 127.0.0.1 port 80 [tcp/http] succeeded!
```

* 扫描指定主机的某个端口段的端口开放信息？？？

```
nc  -v  -z  Target_IP   Target_Port_Start  -  Target_Port_End
```


* 扫描指定主机的某个UDP端口段，并且返回端口信息？？？

```
nc -v   -z  -u  Target_IP  Target_Port_Start   -   Target_Port_End
```

* 扫描指定主机的端口段信息，并且设置超时时间为3秒

```
nc  -vv（-v） -z  -w  time  Target_IP   Target_Port_Start-Targert_Port_End
```


### 端口监听
* 监听本地端口 ？？？

    注：先设置监听（不能出现端口冲突），之后如果有外来访问则输出该详细信息到命令行
    
```
nc  -l  -p  local_Port
```

* 监听本地端口，并且将监听到的信息保存到指定的文件中

```
nc -l  -p local_Port > target_File
```


### 连接远程系统
```
nc Target_IP  Target_Port
```

