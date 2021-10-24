## ifconfig

Linux ifconfig命令用于显示或设置网络设备。

ifconfig可设置网络设备的状态，或是显示目前的设置。

### 语法

```
ifconfig [网络设备][down up -allmulti -arp -promisc][add<地址>][del<地址>][<hw<网络设备类型><硬件地址>][io_addr<I/O地址>][irq<IRQ地址>][media<网络媒介类型>][mem_start<内存地址>][metric<数目>][mtu<字节>][netmask<子网掩码>][tunnel<地址>][-broadcast<地址>][-pointopoint<地址>][IP地址]
```

参数说明：

* add<地址> 设置网络设备IPv6的IP地址。
* del<地址> 删除网络设备IPv6的IP地址。
* down 关闭指定的网络设备。
* <hw<网络设备类型><硬件地址> 设置网络设备的类型与硬件地址。
* io_addr<I/O地址> 设置网络设备的I/O地址。
* irq<IRQ地址> 设置网络设备的IRQ。
* media<网络媒介类型> 设置网络设备的媒介类型。
* mem_start<内存地址> 设置网络设备在主内存所占用的起始地址。
* metric<数目> 指定在计算数据包的转送次数时，所要加上的数目。
* mtu<字节> 设置网络设备的MTU。
* netmask<子网掩码> 设置网络设备的子网掩码。
* tunnel<地址> 建立IPv4与IPv6之间的隧道通信地址。
* up 启动指定的网络设备。
* -broadcast<地址> 将要送往指定地址的数据包当成广播数据包来处理。
* -pointopoint<地址> 与指定地址的网络设备建立直接连线，此模式具有保密功能。
* -promisc 关闭或启动指定网络设备的promiscuous模式。
* [IP地址] 指定网络设备的IP地址。
* [网络设备] 指定网络设备的名称。


### 实例
显示网络设备信息

```sh
# ifconfig        
eth0   Link encap:Ethernet HWaddr 00:50:56:0A:0B:0C 
     inet addr:192.168.0.3 Bcast:192.168.0.255 Mask:255.255.255.0
     inet6 addr: fe80::250:56ff:fe0a:b0c/64 Scope:Link
     UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
     RX packets:172220 errors:0 dropped:0 overruns:0 frame:0
     TX packets:132379 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:1000 
     RX bytes:87101880 (83.0 MiB) TX bytes:41576123 (39.6 MiB)
     Interrupt:185 Base address:0x2024 

lo    Link encap:Local Loopback 
     inet addr:127.0.0.1 Mask:255.0.0.0
     inet6 addr: ::1/128 Scope:Host
     UP LOOPBACK RUNNING MTU:16436 Metric:1
     RX packets:2022 errors:0 dropped:0 overruns:0 frame:0
     TX packets:2022 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:0 
     RX bytes:2459063 (2.3 MiB) TX bytes:2459063 (2.3 MiB)
```

启动关闭指定网卡
```sh
# ifconfig eth0 down
# ifconfig eth0 up
```

给网卡增加一个虚拟(子)IP地址
```sh
ifconfig eth0:8 192.168.9.9/24  
##然后再通过route -n 就会发现多出一条映射管理。
```

为网卡配置和删除IPv6地址
```sh
# ifconfig eth0 add 33ffe:3240:800:1005::2/ 64 //为网卡设置IPv6地址

# ifconfig eth0 del 33ffe:3240:800:1005::2/ 64 //为网卡删除IPv6地址
```

用ifconfig修改MAC地址
```sh
# ifconfig eth0 down //关闭网卡
# ifconfig eth0 hw ether 00:AA:BB:CC:DD:EE //修改MAC地址
# ifconfig eth0 up //启动网卡
# ifconfig eth1 hw ether 00:1D:1C:1D:1E //关闭网卡并修改MAC地址 
# ifconfig eth1 up //启动网卡
```

配置IP地址
```sh
# ifconfig eth0 192.168.1.56 
//给eth0网卡配置IP地址
# ifconfig eth0 192.168.1.56 netmask 255.255.255.0 
// 给eth0网卡配置IP地址,并加上子掩码
# ifconfig eth0 192.168.1.56 netmask 255.255.255.0 broadcast 192.168.1.255
// 给eth0网卡配置IP地址,加上子掩码,加上个广播地址
```

启用和关闭ARP协议
```sh
# ifconfig eth0 arp  //开启
# ifconfig eth0 -arp  //关闭
```

设置最大传输单元
```sh
# ifconfig eth0 mtu 1500 
//设置能通过的最大数据包大小为 1500 bytes
```