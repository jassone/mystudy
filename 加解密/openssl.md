## openssl

### 简介
OpenSSL 是一个开源项目，是**TLS/SSL协议的开源实现，提供开发库和命令行程序**。其组成主要包括一下三个组件：

* openssl：多用途的命令行工具
* libcrypto：加密算法库
* libssl：加密模块应用库，实现了ssl及tls

openssl可以实现：**秘钥证书管理、对称加密和非对称加密**。


### 命令
openssl最为常见的几个指令如下：

* genrsa 生成RSA参数
* req
* x509
* rsa
* ca

### 命令demo
1. 显示SSL握手过程，SSL证书链，公钥证书以及其他相关的状态和属性信息。
openssl s_client -state -connect baidu.com:443