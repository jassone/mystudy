## 系统配置文件sysconfig　

如果一些在此列出的文件没有出现在你的/etc/sysconfig/目录中，可能是相应的程序没有安装的原因。下面将对这些文件进行分别介绍，在此只对这些配置文件进行一般程度的说明，如果要看它们的完整内容，请查看其手册页。

### /etc/sysconfig/amd

此文件的内容是为启用amd守护进程提供它的各种参数，这些参数允许此进程自动挂载或卸载文件系统。

### /etc/sysconfig/apmd

高级电源管理所使用的配置文件。

### /etc/sysconfig/arpwatch

此文件为arpwatch守护进程的配置文件，在系统启动时arpwatch守护进程会使用此文件中的某些区段。arpwatch进程主要用来维护一张MAC地址与IP地址的对应表。
### /etc/sysconfig/authconfig

此文件中的内容为主机在进行认证时使用。在此文件中可能有如下所示的配置项：

```sh
USEMD5=<value>：<value＞可以是如下值中的一个：
　　　yes-使用MD5方式认证；
　　　no-不使用MD5方式认证。

USEKERBEROS=<value>:<value>可以是如下值中的一个：
　　　yes-使用kerberos认证
　　　no-不使用kerberos认证

USELDAPAUTH=<value>:<value>可以是如下值中的一个：
　　　yes-使用LDAP认证
　　　no-不使用LDAP认证
```
### /etc/sysconfig/autofs

此文件用来定义自动挂载设备时使用的选项，包括NFS文件系统、CD－ROMS、DISKTTES或其它媒体，文件内容如下所列：

```sh
LOCALOPTIONS＝"<value>":<value>为字符串格式，用来定义自动挂载规则，缺省为空。

DAEMONOPTIONS＝"<value>":<value>值用来设置卸载设备前的时间长度（以秒为单位），缺省为60S。

UNDERSCORETODOT=<value>:<value>为二进制会下值，用来控制是否将下载线转换为点，例如：auto_home转换为auto.home,缺省为1，即允许。

DISABLE＿DIRECT＝<value>：<value>值为二进制，用来设置是否可以禁止直接连接支持，缺省为1，即允许。
```
### /etc/sysconfig/clock

此文件用来控制解释从系统硬件时钟读取的值，其下确值如下：


```　　UTC＝<value>:<value>可以是下列值中的一个：

　　true或yes-硬件时钟设为universal格式

　　false或no-硬件时钟为本地时间

　　ARC＝<value>:<value>为下列值：

　　true或yes-设置为此值时硬件时钟只用于HRC或HLPHABIOS系统

　　false或no-设置为此值时硬件时钟只用于unix系统

　　SRM＝<value>:<value>为下列值：

　　yes或true－设置为此值时系统时间从1900年开始，此值只有于SRm-based ALPHA系统

　　no或false-设置为此值是为普通UNIX使用

　　ZONE＝＜filename>:<filename>为/usr/share/zoneinfo下的时区文件，如　zone="america/newyork"。
```

### /etc/sysconfig/desktop
当使用运行级别5时，此文件为新用户指定桌面和运行显示管理器。其内容可能如下所示：

```sh
DESKTOP＝"<value>":<value>可以为下列值之一：
　　GNOME-使用GNOME桌面环境
　　KDE－使用KDE桌面环境

DISPLAYANAGER＝"<value>":<value>可以为以下值之一：
　　GNOME－使用GNOME显示管理器
　　KDE－使用KDE显示管理器
　　XDM－使用XDM显示管理器
```

### /etc/sysconfig/devlabel

此文件为设备标签配置文件，不必通过手工编辑设置，可以使用/sbin/devlabel命令来修改其中指定项的值来设置此文件。
### /etc/sysconfig/dhcpd

此文件内容提供一些区段给dhcpd守护进程在系统引导时使用，dhcpd守护进程使用DHCP及BOOTP协议为主机自动分配IP地址。更详细的说明请参考dhcpd的帮助页。

### /etc/sysconfig/exim
此文件允许发送信息给一个或多个客户，如果网络需要可以路由此信息。其内容如下：

```sh
DAEMON=<value>:<value>可为下列值中的一个：
　　　yes-exim将配置监听25端口进来的信息
　　　no-exim不监听25端口

QUEUE＝1h　定义发送时间间隔
```
### /etc/sysconfig/firstboot
此文件为firstboot守护进程的配置文件。
### /etc/sysconfig/gpm
此文件通过其内容中的一些段提供给gpm守护进程在系统引导时使用。gpm进程为鼠标服务，此文件内容包括一些与鼠标相关的如鼠标键数、接口信息等。

### /etc/sysconfig/harddisks
此文件中的内容为系统中已安装的硬盘的参数。其内容如下所列：

```sh
USE_DMA=1：设置硬盘是否使用DMA，值为1使用，0不使用。

MULTIPLE_IO=16：当此项设置值为16时，允许每一次I/O中断读取多个扇区。设置此值可以减少30%－50%的系统开销，但要小心使用此项，缺省为禁止。

EIDE＿32BIT＝3：设置此项值为3时找开（E）IDE32位I/0支持，缺省禁止。

LOOKAHEAD＝1：设置此项为1时允许驱动read-lookahead方式工作，缺省禁止。

EXTRA＿PARAMS＝SPECIFITS：指定EXTRA的参数，缺省为无（没有参数）。
```

### /etc/sysconfig/hwconf
此文件内容列出系统中所有KUDZU检测出来的硬件列表，此文件不应手动编辑，如果对此文件中的内容进行了改变，相应设备就会立即增加或删除。

### /etc/sysconfig/i18n
此文件内容用来设置缺省语言、其它支持的语言和缺省的系统字体，例如：

```sh
LANG="en_us,UTF-8"
SUPPORTED="en_us,UTF-8!en_us:en"
SYSFONT="latareycreb-sun16"
```

### /etc/sysconfig/init
此文件中的内容用来在系统引导期间控制显示和其它功能。它的内容可以为以下所示：

```sh
BOOTUP=<value>:<value>可以为以下值之一：
　　color-设置为此值时，当设备初始化成功或失败时显示不同的颜色。
　　verbise-设置为此值时为一种旧的显示样式，提供一些信息比如成功或失败的信息

RES_COL=<value>:<value＞的内容为以数字方式表示的屏幕显示的信息的列数，缺省为60。

MOVE_TO_COL=<value>:<value>设置的值为移动光标时移动的列数。

ECHO_EN:此命令设定通过用echo -en命令光标移动的行数。

SETCOLOR＿SUCCESS＝＜value>:<value>设置的值用来设定echo -en命令成功时显示的颜色，缺省为绿色。

SETCOLOR＿TAILURE＝<value>:<value>设置的值用来设定echo -en命令错误时显示的颜色，缺省为红色。

SETCOLOR＿WARNING＝<value>:<value>设置的值用来设定echo -en命令警告时的颜色，缺省为黄色。

SETCOLOR＿NORMAL＝<value>:<value>设置的会下用来设定echo -en命令一般模式时的颜色为“NORMAL”。

LOGLEVEL=<value>:<value>设置内核初始化日志记录层次，缺省为3，设置为8时设置记录所有方面的日志信息，包括debugging信息，设置为1时只记录kernel panics信息，syslogd守护进程在启动时会读取此文件。

PROMPT＝<value>:<value>为下列值中之一：
　　yes－设置为此值时允许通过按键来进行交互模式显示启动
　　no－设置为此值时不允许通过按键来进行交互
```

### /etc/sysconfig/ip6tables-config
此文件中保存的内容用来给内核在ip6tables服务启动或设置IPV6过滤规则时使用。不要直接编辑此文件中的内容，除非非常熟悉ip6tables的结构和规则。规则可以通过使用/sbin/ip6tables命令来创建，并保存规则到/etc/sysconfig/ip6tables文件中，保存规则使用此命令：service ip6tables save

一旦保存了所有的规则到/etc/sysconfig/ip6tables文件中，那么当ip6tables服务启动或系统启动时就会通过读取此文件中的规则来设置系统IPV6防火墙。
### /etc/sysconfig/iptables-config
此文件中保存的内容用来给内核在iptables服务启动或设置IPV4过滤规则时使用。不要直接编辑此文件中的内容，除非非常熟悉iptables的结构和规则。规则可以通过使用/sbin/iptables命令来创建，并保存规则到/etc/sysconfig/iptables文件中，保存规则使用此命令：service iptables save

一旦保存了所有的规则到/etc/sysconfig/iptables文件中，那么当iptables服务启动或系统启动时就会通过读取此文件中的规则来设置系统IPV4防火墙。

###/etc/sysconfig/network-scripts 
此文件中保持的是网络配置，里面有很多子文件
![20210926233755.jpg](https://pic.imgdb.cn/item/615094462ab3f51d919c7f72.jpg)

在linux系统中进行网络管理，我们常常使用强大的ifconfig命令。

但ifconfig命令配置的网卡信息，在网卡重启后机器重启后，配置就不存在。要想将上述的配置信息永远的存的电脑里，那就要修改网卡的配置文件了。配置文件中有一个非常重要的成员：/etc/sysconfig/network-scripts/ifcfg-ethx （注：echx是指设备名，例如eth0等）。

在ifcfg-ethx文件配置中的网络信息在重启后仍然生效，在网络管理中相当有用，现在我就ifcfg-ethx文件进行IPV4网络配置方法进行简单的说明。

打开第一张网卡文件

```sh
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0  
DEVICE=eth0  #网卡设备名称   
ONBOOT=yes  #启动时是否激活 yes | no  
BOOTPROTO=static  #协议类型 dhcp bootp none  
IPADDR=192.168.1.90  #网络IP地址  
NETMASK=255.255.255.0  #网络子网地址  
GATEWAY=192.168.1.1  #网关地址  
BROADCAST=192.168.1.255  #广播地址  
HWADDR=00:0C:29:FE:1A:09  #网卡MAC地址  
TYPE=Ethernet  #网卡类型为以太网
```

在修改文件ifcfg-ethx后还需要重新导入文件才能生效，具体命令如下：

```sh
[root@localhost~]# /etc/init.d/network reload   #命令有start | restart | stop | reload
```

## Mac系统下
mac下参考/Library/Preferences/SystemConfiguration里面的内容

配置网络的命令可以使用: networksetup