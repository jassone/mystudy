## http首部

## 一、http请求/响应报文总览
http请求报文
![54DE7988-E11D-4988-AB29-C632F16F07EB.pn](https://pic.imgdb.cn/item/6187893a2ab3f51d91763722.png)

http响应报文
![E1BABACC-3BDE-40E9-B9D3-B2B71F3920B9.png](https://pic.imgdb.cn/item/618789982ab3f51d9176cad7.png)

### 二、4种http首部字段类型
HTTP/1.1规范定义了如下47种首部字段。
1. 通用首部字段
请求报文和响应报文都会用到的首部。

2. 请求首部字段
客户端向服务端发起请求时使用的首部。

3. 响应首部字段
从服务端向客户端返回响应报文时使用的首部。

4. 实体首部字段
针对请求报文和响应报文的`实体部分`使用的首部。

![20180628095643501.png](https://pic.imgdb.cn/item/61878fe82ab3f51d918010da.png)
![20180712145743240.png](https://pic.imgdb.cn/item/618790242ab3f51d91807413.png)
![20180712145818481.png](https://pic.imgdb.cn/item/618790422ab3f51d9180a8ba.png)

非HTTP/1.1首部字段
Cookie,Set-Cookie,Content-Disposition等在其他RFC中定义的首部字段。

### 三、2种缓存相关的首部类型
Http首部字段将定义成缓存代理和非缓存代理的行为，分为2种：

1. 端到端首部(End-to-end Header)
分在此类别中的首部会转发给请求/响应对应的最终接收目标，且必须保存在由缓存生成的响应中，另外归档它必须被转发。

    个人理解：就是这个首部会一直被转发到终点。

2. 逐跳首部(Hop-by-hop Header)
分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不再转发。HTTP/1.1和之后的版本中，如果要使用逐跳首部首部，而需要提供Connection首部字段。

    个人理解：一次性的转发，之后失效。

HTTP/1.1逐跳首部如下：
* Connection
* Keep-Alive
* Proxy-Authenticate
* Proxy-Authorization
* Trailer
* TE
* Transfer-Encoding
* Upgrade

## 四、通用首部字段详解
### 1 cache-control
* 控制缓存的工作机制。
* Cache-Control:private,no-cache

![c7dfa14ae7b7874.jpg](https://pic.imgdb.cn/item/618799372ab3f51d91901302.jpg)

#### a) 表示是否可缓存的指令
##### public指令
Cache-Control:public

客户端和代理都可以缓存。

##### private指令
Cache-Control:private

客户端可以缓存，代理服务器不能缓存。

##### no-cache指令
Cache-Control:no-cache

![673EC80F-163A-42EA-BCD9-E0401E05F609.png](https://pic.imgdb.cn/item/618a94b72ab3f51d91989893.png)

可以缓存在客户端，但每次都`必须去服务器检查新鲜度`。

客户端发送的请求中如果包含no-cache，则表示`客户端将不会接收缓存过的响应`。于是’中间‘的缓存服务器必须把客户端请求转发给源服务器。

服务端返回的响应中包含no-cache，那么缓存服务器不能对资源进行缓存，源服务器以后也将不再对缓存服务器请求中提出的资源进行有效性确认，且进组其对相应资源进行缓存操作。

#### b)控制可执行缓存的对象的指令
##### no-store指令
Cache-Control:no-store

此指令暗示请求或响应中包含机密信息，不可在本地进行任何缓存请求或响应。

从字面上很容易把no-cache误解为不缓存，事实上no-cache代表不缓存过期的资源(要去服务端验证新鲜度)，缓存会向服务器进行有效期确认后处理资源。no-store才是真正的不进行缓存。

#### c)指定缓存期限和认证的指令
##### s-maxage指令
Cache-Control：s-maxage=604800(单位：秒)

s-maxage指令和max-age指令功能基本相同，不同点在于前者只适用于供多位用户使用的公共缓存服务器（一般指代理），也就是说，对同一用户重复返回响应的服务器来说，这个指令是没有意义的。

另外当使用s-maxage时，则直接忽略对Expires首部字段和max-age指令的处理。
 
##### max-age指令
Cache-Control：max-age=604800(单位：秒)

当客户端发送的请求中包含max-age指令，如果判定缓存资源的缓存时间数值比指定时间的数值更小，那么客户端就能接收缓存的资源。另外，当指定max-age=0时，那么缓存服务器通常需要将请求转发给源服务器。

当服务器返回的响应中包含max-age时，缓存服务器将不对资源有效性再做确认，而max-age数值代表资源保存为缓存的最长时间。

max-age和Expires并存时，优先处理max-age，在HTTP/1.0版本中，与此相反。

##### min-fresh指令
Cache-Control：min-fresh=60(单位：秒)

要求缓存服务器返回至少还未指定时间的缓存资源。比如当min-fresh=60，过了60秒的资源都无法作为响应返回了。

##### max-stale指令
Cache-Control: max-stale=3600（单位：秒）

表示即使缓存过期，但只要过期时长没有超过指定时间，客户端依然会接受缓存。若没有指定时长，则不管过期多长时间，客户端都会接收。

##### only-if-cached指令
Cache-Control: only-if-cached

表示请求使用服务器上的缓存，缓存服务器不必再确认缓存的有效性。若缓存服务器没有缓存，会报504 Gateway Timeout。

##### must-revalidate指令
Cache-Control:must-revalidate

如果使用这条指令，代理服务器会向源服务器再次验证即将返回的响应缓存目前是否仍然有效。若代理服务器无法连通源服务器，会报504GatewayTimeout。
使用must-revalidate会使max-stale失效。

##### proxy-revalidate指令
Cache-Control:proxy-revalidate

要求所有的缓存服务器再接收到客户端带有该指令的请求返回响应之前，必须再次验证缓存的有效性。

##### no-transform指令
Cache-Control:no-transform

表示无论在请求还是响应中，缓存都不能改变实体主体的媒体类型，这可以防止缓存或代理服务器压缩图片等类似操作。

##### Cache-Control扩展
省略

### 2 Connection
两个作用：
1. 控制代理不再转发的首部字段
比如客户端发起请求时带上Connection:Upgrade(不再转发的首部字段名)，则打到代理服务器后，代理服务器会把首部字段Upgrade删除后再转发给你源服务器

2. 管理持久连接

    HTTP/1.1默认链接是持久链接，如果要关闭则设置Connection:close。开启则Connection:Keep-Alive。
    
    客户端若发起了Keep-Alive，服务端也会在响应报文首部字段中加上Connection：Keep-Alive。

### 3 Data
表明创建报文的日期和时间。

### 4 Pragma
Pragma: no-cache

HTTP/1.1之前版本的遗留字段。为了兼容性，可以同时加上Cache-Control: no-cache和Pragma：no-cache。

### 5 Trailer
Trailer会事先说明在报文主体后记录了哪些首部字段，该首部字段可应用在HTTP/1.1版本分块传输编码时。

![A4FC2FED-D2C4-489F-A940-ADA0AFB122D4.png](https://pic.imgdb.cn/item/6187bbb82ab3f51d91c8e317.png)
以上例子中，指定首部字段Trailer的值为Expires,在报文主体之后(分块长度0之后)出现了首部字段Expires。

### 6 Transfer-Encoding
Transfer-Encoding:chunked

指定报文主体编码方式。

https://blog.csdn.net/u014569188/article/details/78912469
### 7 Upgrade
Upgrade:TLS/1.0

用于检测是否可以用更高的协议通信，其参数值可以用来指定一个完全不同的通信协议。

服务器可用101 Switch Protocol状态码返回。

注意：Upgrade对象如果仅限于客户端和邻接服务器之间，需加上Connection：Upgrade，以确保该字段不再被转发。

![127A2CED-4302-4271-B823-4E6BDBFD29D4.png](https://pic.imgdb.cn/item/6187c3522ab3f51d91dd7d30.png)

### 8 Via
![17108100-935d709164b7975f.png](https://pic.imgdb.cn/item/6187c4af2ab3f51d91e0adf4.png)

为了追踪报文的传输路径，可以防止避免回环。
每个代理都会在这个字段中加入自己的信息，然后再进行转发。

Via首部是为了追踪传输路径，所以经常会和TRACE方法一起使用。比如，代理服务器接收到由TRACE方法发送过来的请求(其中Max-Forwards:0)时，代理服务器就不能再转发该请求了，这时代理服务器会将自身的信息附加到Via首部后，返回该请求的响应Trailer。

### 9 Warning
告知用户与缓存相关的问题的警告。
格式：
![17108100-e872326cbfada7a4.png](https://pic.imgdb.cn/item/6187c8ae2ab3f51d91e9ea8d.png)

类型：
![17108100-59f1a8eff74ce269.png](https://pic.imgdb.cn/item/6187c8ca2ab3f51d91ea2b23.png)

## 五、请求首部字段详解
### 1  Accept
![17108100-4f4d94caf3137015.png](https://pic.imgdb.cn/item/6187c9862ab3f51d91ec0a75.png)

* type/subtype中的subtype应该是type的子类型；
* q表示权重，最高为1，默认为1，表示对应的媒体类型的返回倾向；
* 通过设置Accept，可以避免接收浏览器不想接收的类型，比如浏览器不支持PNG，那accept就不指定image/png，而制定image/jpeg等可处理的类型。
* 常用媒体类型：文本类型(text/html等)，图片类型(image/jpeg等)，视频类型(video/mpeg等)，应用程序使用的二进制文件(application/octen-stream等)。

### 2 Accept-Charset
Accept-Charset:iso-8859-5,unicode-1-1;q=0.8

用于告知服务器客户端支持的字符集以及字符集的优先顺序，可以一次指定多个字符集，可以通过通过设置权重q来安排有限顺序。

### 3 Accept-Encoding
Accept-Encoding:gzip,deflate

用于告知服务器客户端支持的内容编码以及编码的有限顺序。即压缩格式。

常用的内容编码：gzip,compress,deflate,identity

### 4 Accept-Language
Accept-Language:zh-cn,zh;q=0.7,en-us,en;q=0.3

告知服务端用户可以处理的自然语言集，以及自然语言集的相对优先级，也可通过q指定优先级。

### 5 Authorization
![17108100-4f246feedb6d601e.png](https://pic.imgdb.cn/item/6187cdae2ab3f51d91f5e8de.png)

用于向服务器发送用户代理的认证信息。

### 6 Expect
![17108100-05be026b6fc89e43.png](https://pic.imgdb.cn/item/6187ce5e2ab3f51d91f759a2.png)

客户端使用Expect来告知服务器，期望服务器出现某种特定的行为。如果服务器服务无法理解客户端发出的期望，会返回状态码417 Expectation Failed.
比如客户端希望服务器返回状态码100 Continue, 发送Expect：100-continue。

### 7 From
From:xxx.jp

告知服务器使用用户的电子邮件地址。

### 8 Host
用户告知服务器请求的资源所处的互联网主机和端口号。
由于一台服务器可能存在多个不同域名的虚拟主机，所以单单依靠ip无法正确区分要访问哪个主机，这就需要Host字段来指出请求的主机名。

是HTTP/1.1规范内唯一一个必须被包含在请求内的首部字段。

### 9 If-Match
附带条件的请求：形如If-xxxx的请求，称之为条件请求，服务器接收到这样的请求后，之有条件判断为真，才会执行请求。

![17108100-a676eeafd47b517c.png](https://pic.imgdb.cn/item/6187d6892ab3f51d91081fe3.png)

If-Match 用于指定想要获取的资源的ETag值，只有在资源的ETag值与之相同时，服务器才会执行请求，否则会返回状态码412 Precondition Failed。
若使用*号指定If-Match，那么服务器就不会对ETag进行比对，直接返回资源。
 
### 10 If-Modified-Since 
![17108100-c44f0643fa0dcecc.png](https://pic.imgdb.cn/item/6187d7352ab3f51d91099a34.png)

客户端指定所要的资源要在某个时间过后要发生更新，如果不符合要求，服务器会返回304 Not Modified错误。

### 11 If-None-Match
与If-Match相反

![17108100-5e14b4a5eec34b7.png](https://pic.imgdb.cn/item/6187d8032ab3f51d910b6c96.png)
 
和If-Modified-Since类似，可以获取最新的资源。

### 12 If-Range
使用If-Range和不使用If-Range的对比

![17108100-419d5165aa7be251.png](https://pic.imgdb.cn/item/6187da632ab3f51d910eeb04.png)

![](https://pic.imgdb.cn/item/6187dc452ab3f51d9112bdf9.png)

If-Range指定所请求的资源的ETag值或者时间，当对应资源的ETag或者时间与之相同时，返回所指定范围的资源，否则的话返回全部资源。
如果不使用If-Range，仍然可以用Range发起范围请求，并用If-Match指定ETag值或时间，但是若对应的资源的ETag值或时间不相同时，会返回状态码412 Precondition Failed，催促客户端再次发起请求来请求全部资源，这样就会花费两倍的时间。

### 13 If-Unmodified-Since
 与If-Modified-Since相反。

### 14 Max-Forwards
用来指定可经过的服务器的最大数目，类似于TTL。
![17108100-f4e3cf28bfc1bdee.png](https://pic.imgdb.cn/item/6187ddae2ab3f51d9114b1d6.png)

可以用Max-Forwards对传输路径的通信状况有所把握,也可以防止陷入循环。

### 15 Proxy-Authorization
响应代理服务器发来的认证质询。
![17108100-61849f8f33a49b26.png](https://pic.imgdb.cn/item/6187df282ab3f51d911608c8.png)

### 16 Range
用于只需要获取部分资源的范围请求。服务器如果成功处理范围请求，会返回206 Partial Content响应，否则返回状态码200 OK和全部资源。

### 17 Rerferer
 ![17108100-f54f0a016b0d7c6a.png](https://pic.imgdb.cn/item/6187e00f2ab3f51d91169d6a.png)
 
### 18 TE
![17108100-baac024616f11320.png](https://pic.imgdb.cn/item/6187e0b82ab3f51d911702fb.png)

### 19 User-Agent

## 六、响应首部字段详解
### 1 Accept-Ranges
Accept-Ranges：bytes(可以处理范围请求)

服务器用来告诉客户端自身能否处理范围请求。当不能处理范围请求时，Accept-Ranges：none

### 2 Age
![17108100-a01cb7d81fbacaae.png](https://pic.imgdb.cn/item/6187e5942ab3f51d911c0863.png)

Age字段的定义：当代理服务器用自己缓存的实体去响应请求时，用该头部表明该实体从产生到现在经过多长时间了。

### 3 ETag(Entity Tags)
![17108100-a54bc4d9bb9a8c07.png](https://pic.imgdb.cn/item/6187e6792ab3f51d911d43d1.png)

首部字段ETag以字符串的方式告知客户端资源的实体标识。服务器会对每一份资源作相应的唯一ETag标识。

当资源更新时，ETag值也要相应更新，生成值没有统一算法，仅由服务器来支配。

ETag有强弱之分
* 强ETag：无论实体发生了多么微小的变化都会改变其值。
* 弱ETag：只用于提示资源是否相同，只有发生了根本改变才会改变ETag值。字段开始处会附加W/(Etag:W/"12344")。

### 4 Location
将客户端重新引导至某个资源。配合3XX状态码使用。

### 5 Proxy-Authenticate
![17108100-258d77e95a773.png](https://pic.imgdb.cn/item/6187e7f32ab3f51d91200c7e.png)
把代理服务器所要求的认证信息发送给客户端。

### 6 Retry-After
Retry-After：60(秒)/也可以是具体日期时间
告知客户端发起下次请求的间隔时间。
主要配合状态码503响应，或3XX响应使用。

### 7 Server
Server:Apache/2.2.17(Unix)
告知客户端安装在服务器上的HTTP服务器应用程序的信息。

### 8 Vary
源服务器用Vary向代理服务器传达缓存使用方法的命令。

![17108100-57ea0d6fbe6c19d.png](https://pic.imgdb.cn/item/618890b82ab3f51d919665dc.png)
这里以Accept-Language为例:
当客户端请求中的Accept-Language字段与缓存相同时才可返回缓存，否则缓存服务器就要向源服务器重新获取资源。

### 9 WWW-Authenticate
![17108100-2280050b48e3995a.png](https://pic.imgdb.cn/item/618891452ab3f51d9196cb5e.png)

## 七、实体首部字段详解
### 1 Allow
![17108100-9066747e963aa0d7.png](https://pic.imgdb.cn/item/618892992ab3f51d9197ce8d.png)

* 用来告知服务器URL所指定资源的支持的所有HTTP方法。
* 当服务器接收到不支持的HTTP方法时，会返回405 Method Not Allowed方法，与此同时，还会把所有支持的HTTP方法写在Allow中返回。

### 2 Content-Encoding
Content-Encoding:gzip

服务器用来告知客户端对实体信息选用的编码方式，内容编码是指在不丢失实体信息的前提下进行的。数值同Accept-Encoding。

### 3 Content-Language
实体主体使用的自然语言。

### 4 Content-Length
Content-Length:1500(字节)
表明实体主体的大小，单位为字节。

Content-Length与实际消息长度不一致, 程序可能会发生比较奇怪的异常, 如:

* Content-Length比实际的长度大, 服务端/客户端读取到消息结尾后, 会等待下一个字节, 自然会无响应直到超时。
* 请求被截断, 导致下一个请求解析出现错乱.

Content-Length首部指示出报文中实体主体的字节大小. 这个大小是包含了所有内容编码的, 比如, 对文本文件进行了gzip压缩的话, Content-Length首部指的就是`压缩后的大小而不是原始大小`.

### 5 Content-Location
Content-Location:http://baidu.com/index.html

指出返回资源所对应的URL。
比如，当返回的对象与所请求的不同时，首部字段Content-Location会写明URL。（访问http://baidu.com/,返回对象却是http://baidu.com/index.html）

### 6 Content-MD5
![17108100-d07003c627c1ef24.png](https://pic.imgdb.cn/item/61889ab12ab3f51d919e8b0a.png)

字段中的值是由MD5算法根据报文主体内容生成的值，目的在于内容是否在传输中保持完整。算法的结果是二进制数，而HTTP首部无法记录二进制数，所有需以Base64编码进行再次处理。
这种方法并不能确保安全，因为篡改了内容后照样可以再次生成对应的字段值。

### 7 Content-Range
![17108100-83c8ee2b42de5dbd.png](https://pic.imgdb.cn/item/61889b8b2ab3f51d919f2a17.png)

### 8 Content-Type
Content-Type:text/html;charset=UTF-8

指出实体主体内对象的媒体类型，还可以说明采用的字符集。
 
### 9 Expires
![17108100-550b9fc9dfffe370.png](https://pic.imgdb.cn/item/61889c682ab3f51d919fc334.png)

用来告知资源失效的日期。
当缓存服务器接收到含有Expires的响应后，在Expires时间内，缓存会一直被保留，当超过指定的时间后收到对资源的请求时会向源服务器再次请求资源。
如果源服务器不希望缓存服务器对资源进行缓存，最好把Expires字段内的值设为与Date相同。
max-age存在时，会忽略Expires。

### 10 Last-Modified 
指明资源最后的修改时间。

## 八、为Cookie服务的首部字段
### 1 Set-Cookie
 ![17108100-ff098a83dd07ca9e.png](https://pic.imgdb.cn/item/6188d9602ab3f51d91103461.png)
 
* expires字段指明Cookie的有效期，省略expires时，有效期一般仅限于浏览器程序被关闭之前。可以通过覆盖Cookie来实现Cookie的删除操作，不存在显式的删除操作；
* 使用HttpOnly时，JavaScript的document.cookie就无法读取Cookie的内容了。主要是放在跨站脚本攻击(XSS)。
 
### 2 Cookie
传递给服务端的cookie。

## 九、其它首部字段
Http首部字段是可以自行扩展的，因此会出现一些非标准首部字段。
### 1 X-Frame-Options
![17108100-a56a8c7e6b901eb3.png](https://pic.imgdb.cn/item/6188e20d2ab3f51d91303410.png)

### 2 X-XSS-Protection
![17108100-e0e46c765d1f9ddc.png](https://pic.imgdb.cn/item/6188e2cc2ab3f51d91335b2c.png)

### 3 DNT
![17108100-6a1025d5e444c96b.png](https://pic.imgdb.cn/item/6188e35a2ab3f51d91347a4c.png)

### 4 P3P
![17108100-1afa8d0b3e4a0f8c.png](https://pic.imgdb.cn/item/6188e3ee2ab3f51d91359192.png)




