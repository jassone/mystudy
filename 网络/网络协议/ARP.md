## ARP协议
### ARP简介
是一种解决地址问题的协议。以目标IP地址为线索，用来定位下一个应该接收数据分包的网络设备对应的MAC地址。

**网络链路层的mac地址一直在变。**

### ARP原理
简单点说，ARP是借助ARP请求与ARP相应两种类型的包确定MAC地址的。

查看本机ARP列表，其中第一个就是目的主机(当前路由器)MAC地址。
```sh
➜  / arp -a
? (192.168.1.1) at 60:7e:cd:99:3a:67 on en0 ifscope [ethernet]
? (224.0.0.251) at 1:0:5e:0:0:fb on en0 ifscope permanent [ethernet]
? (239.255.255.250) at 1:0:5e:7f:ff:fa on en0 ifscope permanent [ethernet]
```

> arp -d 可删除mac映射，但是内核会很快请求一下，再次拿到映射关系。


### ARP流程

为了演示删除APR映射表后，后续的具体流程，我们这样操作：

第一步，将监控打开。
```sh
➜  / sudo tcpdump -nn -i en0 port 80 or arp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on en0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

第二步，删除arp列表的同时请求百度。 
```sh
sudo arp -d 192.168.1.1 && curl www.baidu.com
```

##### 接上面tcpdump打印出扫码的结果
```sh
09:20:25.107973 ARP, Request who-has 192.168.1.1 tell 192.168.1.8, length 28
09:20:25.110245 ARP, Reply 192.168.1.1 is-at 60:7e:cd:99:3a:67, length 28
09:20:25.110492 IP 192.168.1.8.55277 > 36.152.44.95.80: Flags [S], seq 5185057, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 1932430422 ecr 0,sackOK,eol], length 0
09:20:25.122801 IP 36.152.44.95.80 > 192.168.1.8.55277: Flags [S.], seq 2016209365, ack 5185058, win 8192, options [mss 1412,nop,wscale 5,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,sackOK,eol], length 0
09:20:25.122899 IP 192.168.1.8.55277 > 36.152.44.95.80: Flags [.], ack 1, win 4096, length 0
09:20:25.123049 IP 192.168.1.8.55277 > 36.152.44.95.80: Flags [P.], seq 1:78, ack 1, win 4096, length 77: HTTP: GET /  
```

### ARP 属于哪一层
ARP协议划到网络层，是因为ARP协议属于TCP/IP协议簇。在TCP/IP模型中，所有定义的协议至少是在网际层以上的。但按照OSI的标准，数据向下传递时每层会加上自己的信息。当网络层的IP包进入链路层时，链路层通过ARP协议添加链路头，这显然不是网络层的功能。所以有很多人说ARP是链路层的。

> 在OSI模型中ARP协议属于链路层。
> 而在TCP/IP模型中，ARP协议属于网络层。