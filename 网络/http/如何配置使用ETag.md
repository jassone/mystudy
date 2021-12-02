## ETag配置和使用

## 一、服务端代码层面实现

```php
$str = "Barret Lee";
$Etag = md5($str);
 
if(array_key_exists('HTTP_IF_NONE_MATCH', $_SERVER) and $_SERVER['HTTP_IF_NONE_MATCH'] == $Etag){
    header("HTTP/1.1 304 Not Modified");
    exit();
} else {
    header("Etag:" . $Etag);
    echo $str;
}
```


更通用的方法
```php
function fsdk_http_etag() {
    ob_start('enable_etag');
}

static function enable_etag($buffer, $flags = 1) {
    if ($flags && $buffer) {
        $etag = md5($buffer);
//            header('Last-Modified: '. gmdate('D, d M Y H:i:s', time()) . ' GMT');
        header('Etag:' . $etag);
//            // @todo:
//            if ($_SERVER['HTTP_IF_NONE_MATCH'] === $etag) {
//                header('HTTP/1.1 304 Not Modified');
//                exit;
//            }
    }
    // ob_start('fsdk_http_etag')
    // 如果 output_callback 返回 FALSE ，其原来的输入 内容被直接送到浏览器。
    return false;
}
```

## 二、服务端服务器配置实现
### 1、nginx
1) 确认Nginx版本，命令：Nginx安装目录/sbin/nginx–v，版本为1.7.3及更高，继续步骤2；版本为1.7.3以下，1.3.3及以上，进行步骤3；版本为1.3.3以下，不支持ETag，请升级您的Nginx。

2) 确认没有关闭ETag：打开Nginx的配置文件nginx.conf(默认位置Nginx安装目录/conf/)，确保其中没有出现etag off;，下图为出现的情况，请将此行删除。

```sh
http {
    etag off;
    ...
}
```

　　确认没有使用ngx_headers_more清除ETag头：同样在配置文件中不能出现如下语句的任意一句，如果出现请将其删除。
　　
```sh
more_set_headers -s 404 -t 'ETag';
more_clear_headers 'Etag';
```

　　重新启动Nginx，就启用ETag功能了。

3) 查看是否开启了gzip，且是否和etag出现冲突，出现冲突去步骤4，没有去步骤2。

　　打开Nginx的配置文件nginx.conf(默认位置Nginx安装目录/conf/)，其中出现gzip on；语句证明开启了gzip，如图
```sh
http {
    gzip on;
    ...
}
```

　　开启gzip时，可能与etag出现冲突，用浏览器多次请求此网站的静态元素，如果只返回200，不返回304，证明存在冲突，请去步骤4；没有冲突去步骤2。

4、请关闭gzip，即将上一步中的gzip on;改为gzip off；然后去步骤2。

### 2、apache

1) 在要启用 ETag 的目录下增加.htac­cess 文件并在其中增加一行：

```sh
FileETag MTime Size
```
以覆盖默认的 Innode MTime Size 的 ETag，因为默认的 ETag 使用到的 Inn­ode 会导致相同的文件在分布式服务器上产生的 ETag 不同。

2) 如果.htacces 文件已经存在，请确保要启用 ETag 的目录 /.htac­ces 文件中没有 FileETagNone。如果存在 FileETag None，请删去该行。

检查没有用 mod_headers 将 ETag 除去，即 httpd.conf 文件中没有出现下面的语句，

```
LoadModule headers_module modules/mod_headers.so
Header unset ETag
```
如果请删除 Headerunset ETag 这一行。

3) 重新启动 httpd，就启用 ETag 了。

