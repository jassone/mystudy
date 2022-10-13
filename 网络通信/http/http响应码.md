## HTTP响应码
## 一、总览
|  | 类型 |说明|
| --- | --- |--- |
| 1XX | informational(信息性状态码)|接收的请求正在处理|
| 2XX | success(成功状态码) |请求正常处理完毕|
| 3XX| redirection（重定向状态码） |需要进行附加操作以完成请求|
|4XX| client error（客户端错误状态码） |服务器无法处理请求|
| 5XX |  server error（服务端错误状态码）|服务器处理请求出错|

## 二、常用状态码

### 1、2XX
* 200:成功
* 204：No Content，服务端返回的报文中不含有实体的主体部分，页面不发生更新。一般在只需要从客户端往服务端发送消息，而对客户端不需要发生内容时使用。
* 206:Partial Content，表示客户端进行了范围请求，而服务端成功执行了这部分的GET请求。响应报文中包含由Content-Range指定范围的实体内容。

### 2、3XX

* 301:Moved Permanently,永久性重定向。该状态码表示请求的资源已经被分配到新的URI。之前的URI失效。所以保存的书签中建议改为新的URI。比如之前某个域名不再使用，需要跳转到新域名。**浏览器会缓存。**
* 302:Found,临时重定向,**以后可能还会变化**。比如未登录时先跳转登陆页面，再跳转到原页面。**浏览器不会缓存。**
* 303:See Other,由于请求对应的资源存在另一个URI，应该使用`GET`方法定向获取请求的资源。

    303 状态码和 302 Found 状态码有着相同的功能，但 303 状态码明确表示客户端应当采用 GET 方法获取资源，这点与 302 状态码有区别。
    
    比如，当使用 POST 方法访问 CGI 程序，其执行后的处理结果是希望客户端能以 GET 方法重定向到另一个 URI 上去时，返回 303 状态码。虽然 302 Found 状态码也可以实现相同的功能，但这里使用 303 状态码是最理想的。

    当 301、302、303 响应状态码返回时，`几乎所有的浏览器都会把POST改成GET`，`并删除请求报文内的主体`，之后请求会自动再次发送。
    
    301、302 标准是禁止将POST方法改变成GET方法的，但实际使用时大家都会这么做。

* 304:Not Modified,表示客户端发送附带条件的请3求时，服务端允许请求访问资源，但因发生请求未满足条件的情况后，直接返回304(服务端资源未改变，可直接使用客户端未过期的缓存)。304状态返回时，不包含任何响应的主体部分。

    附带条件是指采用GET请求报文中包含If-Match,If-Modified-Since,If-None-Match,If-Range,If-Unmodified-Sine中任一首部。
    
* 307:Temporary Redirect,临时重定向。307不会从POST变成GET。但是对于处理响应时的行为，每个浏览器可能处理情况不同。

### 3、4XX
* 400：Bad Request，表示请求报文中有语法错误。
* 401：Unauthorized，表示发送的请求需要通过HTTP认证的认证信息(BASIC认证、DIGEST认证)。
服务端代码demo
    ```php
if (!isset($_SERVER['PHP_AUTH_USER'])) {
    header('WWW-Authenticate: Basic realm="My Realm"');
    header('HTTP/1.0 401 Unauthorized');
    echo 'Text to send if user hits Cancel button';
    exit;
} else {
    echo "<p>Hello {$_SERVER['PHP_AUTH_USER']}.</p>";
    echo "<p>You entered {$_SERVER['PHP_AUTH_PW']} as your password.</p>";
}
```

* 403:Forbidden,表明对请求资源的访问被服务器拒绝了。未获得文件系统的访问权限，访问权限出现某些问题都可能触发403。

* 404:Not Found。

### 4、5XX
* 500：Server Error。
* 502: Bad Gateway	作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应。

    `一般是繁忙进程造成的`，如`高并发`中。这里服务器以nginx举例。
    
    将请求提交给网关如php-fpm执行，但是由于某些原因没有执行完毕导致php-fpm进程终止执行。说到此，这个问题就很明了了，与网关服务如php-fpm的配置有关了。

    php-fpm.conf配置文件中有两个参数就需要你考虑到，分别是max_children和request_terminate_timeout。

    1，`max_children`最大子进程数，在高并发请求下，达到php-fpm最大响应数，后续的请求就会出现502错误的。可以通过netstat命令来查看当前连接数。

    2，`request_terminate_timeout`设置单个请求的超时终止时间。设置过小。
    
    3，还应该注意到php.ini中的`max_execution_time`参数。当请求终止时，也会出现502错误的。

    4，当积累了大量的php请求，你重启php-fpm释放资源，但一两分钟不到，502又再次呈现，这是什么原因导致的呢？ 
    
    这时还`应该考虑到数据库`，查看下数据库进程是否有大量的locked进程，`数据库死锁导致超时`，前端终止了继续请求，但是SQL语句还在等待释放锁，这时就要重启数据库服务了或kill掉死锁SQL进程了。
    
* 503：Server Unavailavle，表明服务器暂时处于超负载或正在进行停机维护，现在无法请求。

    大多是因为网站访问量大，造成了流量超限或者并发数大引起的资源超限出现的错误。
    
* 504:Gateway Time-out，充当网关或代理的服务器，未及时从远端服务器获取请求，

    1，客户发送的请求在限定的时间内没有得到相应，过多的请求导致“504 Gateway Time-out”。
    2，也有可能是服务器带宽问题。
    3，nginx方面，504错误一般是与nginx.conf配置有关了。主要与以下几个参数有关：fastcgi_connect_timeout、fastcgi_send_timeout、fastcgi_read_timeout、fastcgi_buffer_size、fastcgi_buffers、fastcgi_busy_buffers_size、fastcgi_temp_file_write_size、fastcgi_intercept_errors。特别是前三个超时时间。如果fastcgi缓冲区太小会导致fastcgi进程被挂起从而演变为504错误。

