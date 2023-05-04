## iptables

iptables是linux系统中非常出名的防火墙软件，但是其实iptables并不是真正的防火墙，我们可以把它理解为一个代理程序，我们通过iptables这个代理程序，编写相应的规则，并将这些规则执行到“安全框架”中去，所以，这个安全框架才是真正的防火墙，linux中这个安全框架叫“netfilter”,netfilter工作在操作系统的内核空间，iptables则工作在系统的用户空间。
netfilter/iptables是linux平台下的“包过滤型防火墙”，是免费的，它可以代替昂贵的商业防火墙解决方案，完成数据包的过滤，网络地址的转换（NAT）等功能。


### 1、拒绝对本机端口8080-8081端口的TCP协议访问：
```sh
iptables -I INPUT -p tcp --dport 8080:8081 -j DROP
```
### 2、拒绝对本机端口843端口的UDP访问：
```sh
iptables -I INPUT -p udp --dport 843 -j DROP
```
### 3、拒绝转发目的端口843的UDP包：
```sh
iptables -I FORWARD -p udp --dport 843 -j DROP
```
