## ssl

1. 生成私钥:

```
openssl genrsa -out nginx.key 2048
```

1. 生成自签发证书:

复制

```
openssl req -new -key nginx.key -out nginx.csr
```

这将提示您输入一些证书信息,如国家/地区、州/省、城市等。请根据实际情况填写。

1. 生成自签发的 X.509 证书:

复制

```
openssl x509 -req -days 365 -in nginx.csr -signkey nginx.key -out nginx.crt
```

这将生成一个有效期为365天的自签发证书。

1. 将生成的 `nginx.key` 和 `nginx.crt` 文件复制到 Nginx 的证书目录中,通常是 `/etc/nginx/ssl/`。
2. 在 Nginx 配置文件中添加以下设置:

复制

```
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    # 其他配置...
}
```