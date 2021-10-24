## curl

### -v

1. 参数输出通信的整个过程，用于调试。

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

