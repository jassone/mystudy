## Apache中https使用自签名证书

## 一、相关依赖

- openssl
- mod_ssl.so

## 二、申请证书或自己生成自签名证书

1. 生成2048位的加密私钥
```sh
➜  git:(master) ✗ sudo openssl genrsa -out server.key 2048
Generating RSA private key, 2048 bit long modulus
..................................................................................+++
...................................................................................................................+++
e is 65537 (0x10001)
➜  桌面 git:(master) ✗ 
```

2. 生成证书签名请求（CSR），这里需要填写许多信息，如国家，省市，公司等
```sh
➜  git:(master) ✗ sudo openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
Country Name (2 letter code) [AU]:zh
State or Province Name (full name) [Some-State]:sh
Locality Name (eg, city) []:sh
Organization Name (eg, company) [Internet Widgits Pty Ltd]:fanli
Organizational Unit Name (eg, section) []:fanli
Common Name (e.g. server FQDN or YOUR name) []:www.xxx.com
Email Address []:123456@qq.com
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:123456
An optional company name []:fanliwang
```

    注意www.xxx.com一定要和httpd.conf文件中的ServerName www.xxx.com保持一致，否则会报错

3. 生成类型为X509的自签名证书。有效期设置3650天，即有效期为10年
```sh
git:(master) ✗ sudo openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=/C=zh/ST=sh/L=sh/O=fanli/OU=fanli/CN=www.xxx.com/emailAddress=123456@qq.com
Getting Private key
```

4. 复制秘钥和证书文件到指定位置
```sh
cp server.crt server.key /usr/local/apache2/conf
```

5. 修改httpd-ssl.conf文件指定相关秘钥和证书位置
```ini
[root@www ~]# vim /usr/local/apache2/conf/extra/httpd-ssl.conf

<VirtualHost _default_:443>

DocumentRoot "/usr/local/apache2/htdocs"

ServerName www.awstats.com:443

ServerAdmin you@example.com

ErrorLog "/usr/local/apache2/logs/error_log"

TransferLog "/usr/local/apache2/logs/access_log"

SSLEngine on

SSLCertificateFile "/usr/local/apache2/conf/server.crt"

SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
----------------------省略若干

</VirtualHost>
```
6. 修改Apache主配置文件并开启相关模块
```ini
[root@www ~]# vim /usr/local/apache2/conf/httpd.conf

打开相关的注释，启用需要的模块

LoadModule rewrite_module modules/mod_rewrite.so

LoadModule ssl_module         modules/mod_ssl.so

LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

修改主机名

ServerName www.awstats.com

下面的需要添加进来

RewriteEngine on

RewriteCond %{SERVER_PORT} !^443$

RewriteCond %{REQUEST_URI} !^/tz.php

RewriteRule (.*) https://%{SERVER_NAME}/$1 [R]
```
7. 重启apache 