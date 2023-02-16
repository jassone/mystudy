## pfctl

todo

[iptables](https://so.csdn.net/so/search?q=iptables&spm=1001.2101.3001.7020)是Linux下的防火墙，可以进行数据包的过滤，在网络层进行数据的转发、拦截或丢弃等，使用非常普遍，功能也非常强大。但是Mac下没有iptables，为了实现流量转发和过滤，要使用到Mac自带的PFctl。PFctl即control the packet filter，是Unix LIKE系统上进行TCP/IP流量过滤和网络地址转换的系统，也能提供流量整形和控制等，详情可以见PF防火墙。

 数据转发分两种情况，流入数据的转发和流出数据的转发。

https://blog.csdn.net/weixin_36345268/article/details/116883309