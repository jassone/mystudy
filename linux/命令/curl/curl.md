## curl

### -v

1. **参数输出通信的整个过程，用于调试。**

```sh
curl -v https://www.baidu.com
```

demo
```sh
➜  git:(master) ✗ curl -v https://www.baidu.com
*   Trying 36.152.44.96...
* TCP_NODELAY set
...
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):

...
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use http/1.1
...
*  subjectAltName: host "www.baidu.com" matched cert's "*.baidu.com"
*  issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign Organization Validation CA - SHA256 - G2
*  SSL certificate verify ok.
> GET / HTTP/1.1
> Host: www.baidu.com
```

### -trace
1. 也可以用于调试，还会输出原始的二进制数据。

```sh
curl --trace - https://www.baidu.com
```

### 不适用代理

```
curl -v --noproxy '*' https://www.google.com
```

### 其他

```sh
#拷贝网络文件
curl https://raw.githubusercontent.com/gin-gonic/examples/master/basic/main.go > main.go
```

## 相关报错

### 1、Stream error in the HTTP/2 framing layer

这个错误通常是由于HTTP/2协议中的帧（Frame）处理或传输中出现问题，也可能是服务器对HTTP/2的实现存在问题，或者是网络连接不稳定。

解决这个问题的方法有几个，你可以尝试下：

1. **使用HTTP/1.1:** 使用HTTP/1.1版本而非HTTP/2。在你的curl命令中加上 --http1.1 参数。像这样：
   ```
   curl --http1.1 https://example.com
   ```
   这将使curl使用HTTP/1.1版本进行请求。

2. **升级Curl:** HTTP/2协议的支持和实现在早期的Curl版本中可能并不完善，升级Curl到最新版本可能修复这个问题。

3. **检查网络连接:** 如果你用的是不稳定的网络连接，可能会出现这个错误。确保你的网络连接稳定。

4. **与服务器管理员联系:** 如果你不能把这个问题解决，问题可能出在服务器的HTTP/2实现上，你可能需要联系服务器的管理员。

**一般第一种方法就可以解决问题**

## 相关wiki

详解 https://www.cnblogs.com/duhuo/p/5695256.html
