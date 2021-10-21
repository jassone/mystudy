## netstat
netstat 是一个告诉我们系统中所有 tcp/udp/unix socket 连接状态的命令行工具。它会列出所有已经连接或者等待连接状态的连接。 该工具在识别某个应用监听哪个端口时特别有用，我们也能用它来判断某个应用是否正常的在监听某个端口。


### 查看tcp连接
* netstat -tunlp

如果是mac
* netstat -tunlp tcp
 
### 查看某个端口号
* netstat -tunlp | grep 8000

如果是mac
* 查询inet：netstat -navf inet | grep 80
* 查询TCP：netstat -navp tcp | grep 80
* 查询UDP：netstat -navp udp | grep 80


### 查看网络路由表
* netstat -rn

```sh
➜  / netstat -rn
Routing tables

Internet:
Destination        Gateway            Flags        Netif Expire
default            192.168.1.1        UGScg          en0       
127                127.0.0.1          UCS            lo0       
127.0.0.1          127.0.0.1          UH             lo0       
169.254            link#5             UCS            en0      !
192.168.1          link#5             UCS            en0      !
192.168.1.1/32     link#5             UCS            en0      !
192.168.1.1        60:7e:cd:99:3a:67  UHLWIir        
224.0.0.251        1:0:5e:0:0:fb      UHmLWI         en0       
239.255.255.250    1:0:5e:7f:ff:fa    UHmLWI         en0       
255.255.255.255/32 link#5             UCS            en0      !

Internet6:
Destination                             Gateway                         Flags         Netif Expire
default                                 fe80::1%en0                     UGcg            en0       
default                                 fe80::%utun0                    UGcIg         utun0       
default                                 fe80::%utun1                    UGcIg         utun1       
::1                                     ::1                             UHL             lo0       
2409:8a1e:1090:6dab::/64                link#5                          UC              en0       
2409:8a1e:1090:6dab:100d:35b4:5737:a8dd ac:bc:32:8d:e3:af               UHL             lo0       
2409:8a1e:1090:6dab:e9fc:786a:4918:7cf2 ac:bc:32:8d:e3:af               UHL             lo0       
fe80::%lo0/64                           fe80::1%lo0                     UcI             lo0       
fe80::1%lo0                             link#1                          UHLI            lo0       
fe80::%en0/64                           link#5                          UCI             en0       
fe80::%llw0/64                          link#12                         UCI            llw0         
fe80::%utun1/64                         fe80::ea8d:b164:afeb:d917%utun1 UcI           utun1       
fe80::ea8d:b164:afeb:d917%utun1         link#14                         UHLI            lo0                         
```