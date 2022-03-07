## iptables

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
