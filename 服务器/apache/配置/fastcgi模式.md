## fastcgi模式配置

## 一、apache 端配置

### 1、mpm 需要改成 event 模式

```
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
#LoadModule mpm_worker_module modules/mod_mpm_worker.so
LoadModule mpm_event_module modules/mod_mpm_event.so
```

### 2、确保启用了 mod_proxy_fcgi

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```

### 3、添加支持 index.php

```
<IfModule php7_module>
	<IfModule dir_module>
		DirectoryIndex index.html index.php
	</IfModule>
</IfModule>
```

### 4、添加 vhost 并 使用 ProxyPassMatch

```
<VirtualHost *:80>                                                      
 ServerAdmin me@com.cn                                           
 DocumentRoot "/usr/local/webdata/www"                                        
 ServerName localhost                    
 <Directory "/usr/local/webdata/www">
    Options FollowSymLinks Includes
    AllowOverride All
    Order allow,deny
    Allow from all                                           
 </Directory>                               
 ProxyRequests Off
 ProxyPassMatch "^/(.*\.php(/.*)?)$" "fcgi://127.0.0.1:9000/usr/local/webdata/www/$1"
</VirtualHost> 
```

**其作用大致是 所有以 .php 结尾的请求都转发给 127.0.0.1:9000 ，fastcig 会一直监听127.0.0.1:9000。**

## 二、php-fpm 配置

/usr/local/etc/php/7.4（本人）
├── php-fpm.conf
├── php-fpm.d
│ └── www.conf
├── php.ini

### 1、www.conf

注意 user group 要与 apache 的 httpd.conf 中的一致
```
[www]
user = _www
group = _www
listen = 127.0.0.1:9000
listen.owner = _www
listen.group = _www
listen.mode = 0666
pm = dynamic
```



