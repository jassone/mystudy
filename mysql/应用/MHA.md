## MHA
MHA（Master HA）是一款开源的 MySQL 的高可用程序，它为 MySQL主从复制架构提供了 automating master failover （自动化主故障转移）功能。MHA 在监控到 master 节点故障时，会提升其中拥有最新数据的 slave 节点成为新的master 节点，在此期间，MHA 会通过于其它从节点获取额外信息来避免一致性方面的问题。MHA 还提供了 master 节点的在线切换功能，即按需切换 master/slave 节点。MHA 能够在30秒内实现故障切换，并能在故障切换中，最大可能的保证数据一致性。目前淘宝也正在开发相似产品 TMHA， 目前已支持一主一从。
　　
当前最成熟的mysql高可用方案。(很好的兼容性)。

### 1、MHA故障发现与转移
![345676543.png](https://pic.imgdb.cn/item/61d97fe02ab3f51d911a4438.png)

* 1、启动：前置检查（配置文件等，不细说）
* 2、运行过程：若master挂了，MHA如何认定需要进行故障转移？
	1、manger每3秒向主节点发送select 1 的SQL语句，判断主节点是否执行。3次ping无反应，认定master异常
	2、避免网络导致的无法ping通，manger让从属服务器MHA node尝试SSH登录检查，若所有node连接不上，从而认定，开始故障转移。

### 2、故障转移：
* 1、切断外界链接（断开虚拟IP跟怀疑挂掉的主机链接），主从同步也断开
* 2、manager使用SSH拉取备份服务器的binlog（复制保存到manager）
* 3、由于binlog跟从服务器的relaylog的数据可能存在不一致，从而进入转移，使得数据保持一致。
* 4、从属服务器之间进行比对，node会检查relaylog哪个是最新的（理解上不管他怎么检查吧），向其他节点发送差异数据并应用。
* 5、这时从属服务器的relaylog一致了，但和主服务器的binlog还可能不一致。这时将旧的master binlog差异的数据发送到从属服务器上并应用。
* 6、数据均一致后，面临以谁为主服务器进行迁移的问题？有三种方案
    1、manger指定谁主
    2、看哪个节点的日志最新
    3、按注册实例列表向后选择（manger中对sql有记录）
* 7、确定谁是主后，从服务器通过changer master命令完成主从链接。
* 8、将原虚拟IP指向新的主服务器
* 9、当旧的主服务器恢复正常，作为从服务器和新的主服务器进行同步（MHA自动完成）

### 3、这个过程存在的细节问题有：（按实际情况解决）
1、binlog不完整
2、迁移丢包，数据不完整
3、旧主服务器跟binlog日志不一致

### 4、优缺点
![12346576543.png](https://pic.imgdb.cn/item/61d979aa2ab3f51d9115aa0e.png)

### 5、相关文档
https://blog.csdn.net/weixin_44907813/article/details/107309347
https://zhuanlan.zhihu.com/p/132508138 