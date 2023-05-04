## nslookup 

todo

nslookup 主要用来诊断域名系统 (DNS) 基础结构的信息。查询DNS的记录，查询域名解析是否正常，在网络故障时用来诊断网络问题

### 1、查询域名解析地址

```sh
nslookup 域名
```

### 2、指定DNS服务器查询

如果我们想获得更权威准确的记录，可以使用指定DNS服务器进行查询。

```sh
语法：nslookup 域名 DNS服务器

示例：nslookup www.sfn.cn ns1.sfn.cn
```

### 3、查询指定解析类型的解析记录

Nslookup可以查询域名指定类型的解析记录，如AAAA记录、CNAME记录、MX记录等。

```sh
语法：nslookup -qt=type 域名
示例：nslookup -qt=AAAA www.163.com
```