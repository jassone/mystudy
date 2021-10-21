TODO

## 一 概述            
ulimit 是一种 linux 系统的内键功能，它具有一套参数集，用于为由它生成的 shell 进程及其子进程的资源使用设置限制。

## 二 相关命令总结

* 查看系统用户所有限制值：ulimit -a
* 查看当前系统打开的文件数量: lsof | wc -l  
* 查看当前进程的打开文件数量：lsof -p pid | wc -l      （lsof -p 1234 | wc -l  ）

* 查看当前进程的最大可以打开的文件数：cat /proc/PID/limits  (如果通过ulimit -n 设置或者修改/etc/security/limits.conf，看看进程是否生效)  
* 查看系统总限制打开文件的最大数量：cat /proc/sys/fs/file-max

## 三 ulimit的功能
limit 用于限制 shell 启动进程所占用的资源（**同时支持硬资源和软资源的限制**），支持以下各种类型的限制：
 
作为临时限制，ulimit 可以作用于通过使用其命令登录的 shell 会话，在会话终止时便结束限制，并不影响于其他 shell 会话。而对于长期的固定限制，ulimit 命令语句又可以被添加到由登录 shell 读取的文件中，作用于特定的 shell 用户。


2、使用ulimit
ulimit 通过一些参数选项来管理不同种类的系统资源。

ulimit 命令的格式为：ulimit [options] [limit]

![2018092819560578.png](https://pic.imgdb.cn/item/61416d172ab3f51d919efbce.png)


主要关注两个：

1）open files：– 用户可以打开文件的最大数目

对应ulimit 的命令ulimit -n，可以使用ulimit -n 临时设置。

对应/etc/security/limits.conf的资源限制类型是：nofile

```sh
* soft nofile 4096    
* hard nofile 4096
```
2）max user processes  – 用户可以开启进程/线程的最大数目

对应ulimit 的命令ulimit  -u  临时修改max user processes的值：ulimit  -u   8192。

对应/etc/security/limits.conf的资源限制类型是：  noproc

```sh
* soft    nproc     8192
```

## 四 ulimit 参数说明
  
| 选项 [options] |含义  |	例子  |
| --- | --- | --- |
| -H | 设置硬资源限制，一旦设置不能增加。 |  ulimit – Hs 64；限制硬资源，线程栈大小为 64K。 |  
| -S | 设置软资源限制，设置后可以增加，但是不能超过硬资源设置。 | ulimit – Sn 32；限制软资源，32 个文件描述符。  |  
| -a | 显示当前所有的 limit 信息。 |ulimit – a；显示当前所有的 limit 信息。   |  
|  -c| 最大的 core 文件的大小， 以 blocks 为单位。 |   ulimit – c unlimited； 对生成的 core 文件的大小不进行限制。|  
| -d |进程最大的数据段的大小，以 Kbytes 为单位。 |  ulimit -d unlimited；对进程的数据段大小不进行限制。|  
| -f | 进程可以创建文件的最大值，以 blocks 为单位。 |  ulimit – f 2048；限制进程可以创建的最大文件大小为 2048 blocks。| 
| -l |  最大可加锁内存大小，以 Kbytes 为单位。|ulimit – l 32；限制最大可加锁内存大小为 32 Kbytes。  | 
| -m | 最大内存大小，以 Kbytes 为单位。 | ulimit – m unlimited；对最大内存不进行限制。 | 
| -n |  可以打开最大文件描述符的数量。| ulimit – n 128；限制最大可以使用 128 个文件描述符。 | 
| -p | 管道缓冲区的大小，以 Kbytes 为单位。 |  ulimit – p 512；限制管道缓冲区的大小为 512 Kbytes。| 
| -s | 线程栈大小，以 Kbytes 为单位。 | ulimit – s 512；限制线程栈的大小为 512 Kbytes。 | 
|-t  |  最大的 CPU 占用时间，以秒为单位。| ulimit – t unlimited；对最大的 CPU 占用时间不进行限制。 | 
| -u | 用户最大可用的进程数。 |  ulimit – u 64；限制用户最多可以使用 64 个进程。| 
|  -v|  进程最大可用的虚拟内存，以 Kbytes 为单位。|ulimit – v 200000；限制最大可用的虚拟内存为 200000 Kbytes。
  | 
   

## 五 使用 ulimit
我们可以通过以下几种方式来使用 ulimit

### 1、在用户的启动脚本中

如果用户使用的是 bash，就可以在用户的目录下的 .bashrc 文件中，加入 ulimit – u 64，来限制用户最多可以使用 64 个进程。此外，可以在与 .bashrc 功能相当的启动脚本中加入 ulimt。

### 2、在应用程序的启动脚本中

如果用户要对某个应用程序 myapp 进行限制，可以写一个简单的脚本 startmyapp。

```sh
ulimit – s 512
myapp
```
以后只要通过脚本 startmyapp 来启动应用程序，就可以限制应用程序 myapp 的线程栈大小为 512K。

### 3、直接在控制台输入 
 
```sh
ulimit – p 256 
```
限制管道的缓冲区为 256K。但只对当前登录shell有效。

### 4、修改所有 linux 用户的环境变量文件：

    vi /etc/profile

    ulimit -u 10000

    ulimit -n 4096

    ulimit -d unlimited

    ulimit -m unlimited

    ulimit -s unlimited

    ulimit -t unlimited

    ulimit -v unlimited

 保存后运行#source /etc/profile 使其生效

### 5、也可以针对单个用户的.bash_profile设置：

```sh
vi ~./.bash_profile

#ulimit -n 1024
```
重新登陆ok

## 六 用户进程的有效范围
ulimit 作为对资源使用限制的一种工作，是有其作用范围的。那么，它限制的对象是单个用户，单个进程，还是整个系统呢？事实上，ulimit 限制的是当前 shell 进程以及其派生的子进程。举例来说，如果用户同时运行了两个 shell 终端进程，只在其中一个环境中执行了 ulimit – s 100，则该 shell 进程里创建文件的大小收到相应的限制，而同时另一个 shell终端包括其上运行的子程序都不会受其影响：

##### Shell 进程 1
ulimit –s 100
cat testFile > newFile
File size limit exceeded

##### Shell 进程 2
cat testFile > newFile
ls –s newFile
323669 newFile
 

##### 针对用户永久生效
通过修改系统的 /etc/security/limits.conf配置文件。该文件不仅能限制指定用户的资源使用，还能限制指定组的资源使用。该文件的每一行都是对限定的一个描述。

limits.conf的格式如下：

```
<domain>                  <type>      <item>     <value> 

username|@groupname       type        resource          limit
```
domain：username|@groupname：设置需要被限制的用户名，组名前面加@和用户名区别。也可以用通配符*来做所有用户的限制。

type：有 soft，hard 和 -，soft 指的是当前系统生效的设置值。hard 表明系统中所能设定的最大值。soft 的最大值不能超过hard的值。用 – 就表明同时设置了 soft 和 hard 的值。

resource：
* core – 限制内核文件的大小
* date – 最大数据大小
* fsize – 最大文件大小
* memlock – 最大锁定内存地址空间
* nofile – 打开文件的最大数目
* rss – 最大持久设置大小
* stack – 最大栈大小
* cpu – 以分钟为单位的最多 CPU 时间
* noproc – 进程的最大数目（系统的最大进程数）
* as – 地址空间限制
* maxlogins – 此用户允许登录的最大数目

要使 limits.conf 文件配置生效，必须要确保 pam_limits.so 文件被加入到启动文件中。

查看 /etc/pam.d/login 文件中有：
session required /lib/security/pam_limits.so

 
例如：解除 Linux 系统的最大进程数和最大文件打开数限制：  

```sh
vi /etc/security/limits.conf  
# 添加如下的行  
* soft noproc 20000  #软连接   
* hard noproc 20000  #硬连接  
* soft nofile 4096    
* hard nofile 4096  
```       
说明：* 代表针对所有用户，noproc 是代表最大进程数，nofile 是代表最大文件打开数

需要注意一点：/etc/security/limits.d下也有noproc最大进程参数的限制。
即 /etc/security/limits.d/下的文件覆盖了/etc/security/limits.conf设置的值 
 
现在已经可以对进程和用户分别做资源限制了，看似已经足够了，其实不然。很多应用需要对整个系统的资源使用做一个总的限制，这时候我们需要修改 /proc 下的配置文件。/proc 目录下包含了很多系统当前状态的参数。
例如 /proc/sys/kernel/pid_max，/proc/sys/net/ipv4/ip_local_port_range 等等，从文件的名字大致可以猜出所限制的资源种类。

注意：
通过读取/proc/sys/fs/file-nr可以看到当前使用的文件描述符总数。另外，对于文件描述符的配置，需要注意以下几点：

* 所有进程打开的文件描述符数不能超过/proc/sys/fs/file-max
* 单个进程打开的文件描述符数不能超过user limit中nofile的soft limit
* nofile的soft limit不能超过其hard limit
* nofile的hard limit不能超过/proc/sys/fs/nr_open

## 相关问题分析（用户进程的有效范围）
**问题1**：su切换用户时提示：Resource temporarily unavailable

或者通过进程跟踪 strace -p pid 看到:Resource temporarily unavailab

通过ulimit -a，得到结果：

```sh
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63463
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65535
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
``` 

在上面这些参数中，通常我们关注得比较多:

open files: 一个进程可打开的最大文件数.

max user processes: 系统允许创建的最大进程数量.

查询
```sh
ps -efL|grep java |wc -l
#24001
```

这个得到的线程数竟然是2万多，远远超过4096

我们可以使用 ulimit -u 20000 修改max user processes的值，但是只能在当前终端的这个session里面生效，重新登录后仍然是使用系统默认值。

正确的修改方式是修改/etc/security/limits.d/20-nproc.conf文件中的值。先看一下这个文件包含什么：

```sh
$ cat /etc/security/limits.d/90-nproc.conf 
# Default limit for number of user's processes to prevent# accidental fork bombs.
# See rhbz #432903 for reasoning.
* soft nproc 8192
```
我们只要修改上面文件中的8192这个值，即可。

**问题2**：linux 打开文件数 too many open files 解决方法
在运行某些命令或者 tomcat等服务器持续运行 一段时间后可能遇到   too many open files。

出现这句提示的原因是程序打开的文件/socket连接数量超过系统设定值。

java进程如果遇到java.net.SocketException: Too many open files，接着可能导致域名解析ava.net.UnknownHostException:

原因是用户进程无法打开系统文件了。

![20181012124449704.png](https://pic.imgdb.cn/item/61440ba92ab3f51d916b9a2c.png)


查看每个用户最大允许打开文件数量

![2018092819560578 (1).png](https://pic.imgdb.cn/item/61416d172ab3f51d919efbce.png)

ulimit -a

其中 open files (-n) 65535 表示每个用户最大允许打开的文件数量是65535 。 默认是1024。1024很容易不够用。

永久修改open files 方法
vim /etc/security/limits.conf  
在最后加入  

```sh
* soft nofile 65535 
* hard nofile 65535  
```

或者只加入

```sh
 * - nofile 65535
```

最前的 * 表示所有用户，可根据需要设置某一用户，例如

```sh
fdipzone soft nofile 8192  
fdipzone hard nofile 8192  
```
注意"nofile"项有两个可能的限制措施。就是项下的hard和soft。 要使修改过得最大打开文件数生效，必须对这两种限制进行设定。 如果使用"-"字符设定, 则hard和soft设定会同时被设定。 
改完后注销一下就能生效。

通过 ulimit -n或者ulimit -a 查看系统的最大文件打开数已经生效了。但此时查看进程的最大文件打开数没有变，原因是这个值是在进程启动的时候设定的，要生效必须重启！

ok，那就重启吧，重启完毕，结果发现依然没变！这奇了怪了，后来经过好久的排查，最终确认问题是，该程序是通过 supervisord来管理的，也就是这进程都是 supervisord 的子进程，而 supervisord 的最大文件打开数还是老的配置，此时必须重启 supervisord 才可以。

当大家遇到limits修改不生效的时候，请查一下进程是否只是子进程，如果是，那就要把父进程也一并重启才可以。

 