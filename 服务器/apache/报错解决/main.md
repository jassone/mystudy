相关报错解决

### 1、 启动

```
httpd -k start

AH00557: httpd: apr_sockaddr_info_get() failed for bianMacBook-Pro.local
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
```
解决：httpd.conf增加配置

```
ServerName localhost:80
```