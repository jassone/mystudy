## 配置
nginx配置可查看配置文档，里面有详细demo说明。

### 1、普通配置

```ini
http {
    server {
        listen       8080;
        server_name  localhost;

        location / {
            root   /Users/bian/webdata/nginx_www;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ \.php$ {
            root           /Users/bian/webdata/nginx_www;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
}
```

### 2、配置https ???

```ini
server {
        listen 443;
        server_name xx.name.com;
        ssl on;

        index index.html index.htm;

        ssl_certificate   cert/215079423330181.cert;
        ssl_certificate_key  cert/215079423330181.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location / {
           root   /public/app/dist;
           index  index.php index.html index.htm;
        }

        location /sell {
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   Host      $http_host;
            proxy_set_header X-NginX-Proxy true;

            proxy_pass         http://127.0.0.1:8080;
            proxy_redirect off;
        }

   }
```

### 3、其他demo配置

```ini
# For more information on configuration, see:

#   * Official English Documentation: http://nginx.org/en/docs/

#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '

                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;


    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    gzip on;
    gzip_static on;
    gzip_min_length 1024;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript   application/x-httpd-php application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;

    gzip_vary off;
    gzip_disable "MSIE [1-6]\.";

    include             /etc/nginx/mime.types;

    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.

        include /etc/nginx/default.d/*.conf;
        
        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

    server {
         listen 443;
         server_name mp.hanxing.store;
         ssl on;
         index index.html index.htm;
         ssl_certificate   cert/cert_mp.hanxing.store.crt;
         ssl_certificate_key  cert/cert_mp.hanxing.store.key;
         ssl_session_timeout 5m;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_prefer_server_ciphers on;

         location / {
            root   /public/sell/app/dist;
            index  index.php index.html index.htm;
         }

         location /sell {
             proxy_set_header   X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header   Host      $http_host;
             proxy_set_header X-NginX-Proxy true;
             proxy_pass         http://127.0.0.1:8080;
             proxy_redirect off;
         }

         error_page 404 /404.html;
              location = /40x.html {

         }

         error_page 500 502 503 504 /50x.html;
            location = /50x.html {
         }
    }
}
```



### 其他

```
    location / {
        client_max_body_size 30M;  // 最大上传文件大小
        proxy_pass http://192.168.2.81:8181;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       # proxy_http_version 1.1;              # 使用 HTTP/1.1 支持 chunked 传输
        proxy_set_header Connection "";      # 防止 Nginx 添加 `Connection: close` 头
        proxy_buffering off;                 # 禁用缓冲，直接将数据流发送到客户端
        proxy_cache off;                     # 禁用缓存
    }
```

