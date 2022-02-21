### 配置

```
// 引入mod_php模块
LoadModule php7_module /usr/local/opt/php@7.4/lib/httpd/modules/libphp7.so

// 匹配到php文件，则认为是PHP脚本，去解析
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

// 默认起始页
<IfModule dir_module>
    DirectoryIndex  index.php index.html
</IfModule>

//host
ServerName localhost:80
```