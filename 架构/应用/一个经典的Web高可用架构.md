## 一个经典的Web高可用集群架构
## 一、方案
DNS轮询与多VIP组合解决利率用问题
![kkhgfdrtyui.png](https://pic.imgdb.cn/item/61de6b5f2ab3f51d91483dd9.png)

### 1、技术方面:
* Keepalived 实现监控心跳自动主备(主机/影子主机)切换
* NDS配置两个ip,轮询映射
### 2、设备方面
* 2*Nginx + N*Tomcat + 1*MySQL
* 构建起了**nginx集群，tomcat集群**

### 3、安全层面
* 做好DNS劫持的预防

## 二、如果单独使用DNS轮询访问tomact机子呢
* 只负责IP轮询获取,不保证节点可用* DNS IP列表变更有延时* 外网IP占用严重(需要给每台tomact申请一个ip)