## 使用/配置
1. 监听本机，选择loopback选项

## 过滤(在过滤栏输入相应字符)
1. 过滤抓取某个端口 
```
tcp.port == 9999
```

2. 源地址为192.168.1.102，
```
ip.src ==192.168.1.102 
```

1. 目标地址为192.168.1.102
```
ip.dst==192.168.1.102
``` 
  
1. 抓取http包
```
http
```

1. 抓取https包
```
tls
```
   
 
## 报错
1. 报错：The capture session could not be initiated on interface 'en0' (You don't have permission to capture on that device).

    解决：增加权限 sudo chmod o+r /dev/bpf*


