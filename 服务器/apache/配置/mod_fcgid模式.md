## mod_fcgid模式

```ini
LoadModule fcgid_module modules/mod_fcgid.so
<IfModule mod_fcgid.c>
          AddHandler fcgid-script .fcgi .php
          #php.ini的存放目录
          FcgidInitialEnv PHPRC "D:/PHP"
          # 设置PHP_FCGI_MAX_REQUESTS大于或等于FcgidMaxRequestsPerProcess，防止php-cgi进程在处理完所有请求前退出
          FcgidInitialEnv PHP_FCGI_MAX_REQUESTS 1000
          #php-cgi每个进程的最大请求数
          FcgidMaxRequestsPerProcess 1000
          #php-cgi最大的进程数
          FcgidMaxProcesses 5
          #最大执行时间
          FcgidIOTimeout 120
          FcgidIdleTimeout 120
          #php-cgi的路径
          FcgidWrapper "D:/PHP/php-cgi.exe" .php
          AddType application/x-httpd-php .php
 </IfModule>
```



```ini
#修改DocumentRoot 路径的配置为：
<Directory "D:/WWW"> 
    Options Indexes FollowSymLinks ExecCGI
    Order allow,deny 
    Allow from all
    AllowOverride All
</Directory>
```