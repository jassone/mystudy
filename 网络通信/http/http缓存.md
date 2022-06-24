## http缓存
## 一、前端缓存
前端缓存可分为两大类：`http缓存`和`浏览器缓存`。下面这张图是前端缓存的一个大致知识点：

![4D59CBD9-7E5D-4A27-9A69-0D952660865C.png](https://pic.imgdb.cn/item/618938912ab3f51d91c183ac.png)

## 二、什么是HTTP缓存 ？
http缓存指的是: 当客户端向服务器请求资源时，会先抵达浏览器缓存，如果浏览器有“要请求资源”的副本，就可以直接从浏览器缓存中提取而不是从原始服务器中提取这个资源。

常见的http缓存只能缓存get请求响应的资源，对于其他类型的响应则无能为力，所以后续说的请求缓存都是指GET请求。

http缓存都是从第二次请求开始的。第一次请求资源时，服务器返回资源，并在respone header头中回传资源的缓存参数；第二次请求时，浏览器判断这些请求参数，命中强缓存就直接200，否则就把请求参数加到request header头中传给服务器，看是否命中协商缓存，命中则返回304，否则服务器会返回新的资源。

## 三、为什么要使用HTTP缓存 ？
根据上面的学习可发现使用缓存的好处主要有以下几点：
1. 减少了冗余的数据传输，节省了网费。
2. 缓解了服务器的压力， 大大提高了网站的性能
3. 加快了客户端加载网页的速度

## 四、http缓存的分类
* 根据是否需要重新向服务器发起请求来分类，可分为(强制缓存，协商缓存)。
* 根据是否可以被单个或者多个用户使用来分类，可分为(私有缓存，共享缓存)。 

强制缓存和协商缓存区别
* 强制缓存如果生效，不需要再和服务器发生交互，
* 而协商缓存不管是否生效，都需要与服务端发生交互。

下面是强制缓存和协商缓存的一些对比：
![21832D38-A7F4-41E2-9358-309DB6A274CE.png](https://pic.imgdb.cn/item/61895a872ab3f51d91e8a6b2.png)
  
### 3.1、强制缓存
强制缓存在缓存数据未失效的情况下（即Cache-Control的max-age没有过期或者Expires的缓存时间没有过期），那么就会直接使用浏览器的缓存数据，不会再向服务器发送任何请求。强制缓存生效时，http状态码为200。这种方式页面的加载速度是最快的，性能也是很好的，但是在这期间，如果服务器端的资源修改了，页面上是拿不到的，因为它不会再向服务器发请求了。这种情况就是我们在开发种经常遇到的，比如你修改了页面上的某个样式，在页面上刷新了但没有生效，因为走的是强缓存，所以Ctrl + F5一顿操作之后就好了。 跟强制缓存相关的header头属性有（Pragma/Cache-Control/Expires）

![8B99EAF2-FFA4-4D3A-9232-7CDC78A6D286.png](https://pic.imgdb.cn/item/61895abb2ab3f51d91e8dab5.png)

 
### 3.2、协商缓存
当第一次请求时服务器返回的响应头中没有Cache-Control和Expires或者Cache-Control和Expires过期还或者它的属性设置为no-cache时(即不走强缓存)，那么浏览器第二次请求时就会与服务器进行协商，与服务器端对比判断资源是否进行了修改更新。如果服务器端的资源没有修改，那么就会返回304状态码，告诉浏览器可以使用缓存中的数据，这样就减少了服务器的数据传输压力。如果数据有更新就会返回200状态码，服务器就会返回更新后的资源并且将缓存信息一起返回。跟协商缓存相关的header头属性有（ETag/If-Not-Match 、Last-Modified/If-Modified-Since）请求头和响应头需要成对出现。

![10FC7021-01BD-4066-89CB-37CEAC8D2148.png](https://pic.imgdb.cn/item/61895f902ab3f51d91f19cbf.png)

#### 3.2.1 Last-Modified 和 If-Modified-Since
Last-Modified 和 If-Modified-Since 是 HTTP 1.0 引入的。

##### Last-Modified
当浏览器第一次访问一个资源的时候，服务器会在 Response 、Header 中返回一个 Last-Modified，代表这个资源最后的修改时间。

##### If-Modified-Since
再次请求服务器时，请求头会携带此字段，值为上次请求时服务器返回的 Last-Modified 的值。

服务器收到请求后发现有头 If-Modified-Since 则与被请求资源的最后修改时间进行比对：

* 若资源的最后修改时间大于 If-Modified-Since，说明资源又被改动过，则响应整片资源内容，返回状态码 200 和最新的资源，响应头中携带最新的缓存标识 Last-Modified。
* 若资源的最后修改时间小于或等于 If-Modified-Since，说明资源无新修改，则响应 HTTP 304，告知浏览器继续使用所保存的 cache。

##### 缺陷
* 如果资源更新的速度是秒以下单位，那么该缓存是不能被使用的，因为 If-Modified-Since 只能检查到以秒为最小计量单位的时间差。
* 如果文件是通过服务器动态生成的，那么该方法的更新时间永远是生成的时间，尽管文件可能没有变化，所以起不到缓存的作用。
* 我们编辑了文件，但文件的内容没有改变。服务端并不清楚我们是否真正改变了文件，它仍然通过最后编辑时间进行判断。因此这个资源在再次被请求时，会被当做新资源，进而引发一次完整的响应——不该重新请求的时候，也会重新请求。
* 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形。

为了解决上面服务器没有正确感知文件变化的问题，**Etag 作为 Last-Modified 的补充出现了**。
 
### 3.2.2 Etag 和 If-None-Match
Etag 和 If-None-Match 是一对报文头，属于HTTP 1.1。

ETag 和 If-None-Match 的值是一串 hash 码，代表的是一个资源的标识符，当服务端的文件变化的时候，它的 hash 码会随之改变。

##### Etag
服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。

ETag 又有强弱校验之分，如果 hash 码是以 "W/" 开头的一串字符串，说明此时协商缓存的校验是弱校验的，只有服务器上的文件差异（根据 ETag 计算方式来决定）达到能够触发 hash 值后缀变化的时候，才会真正地请求资源，否则返回 304 并加载浏览器缓存。

##### If-None-Match
再次请求服务器时，通过此字段通知服务器客户段缓存数据的唯一标识。

服务器收到请求后发现有头 If-None-Match 则与被请求资源的唯一标识进行比对：

* 不同，说明资源被改动过，则响应整片资源内容，返回状态码 200。
* 相同，说明资源无新修改，则响应 HTTP 304，告知浏览器继续使用所保存的 cache。

##### 缺陷
Etag 的生成过程需要服务器额外付出开销，会影响服务端的性能，这是它的弊端。

因此启用 Etag 需要我们审时度势：

* Etag 并不能替代 Last-Modified，它只能作为 Last-Modified 的补充和强化存在。
* Etag 在感知文件变化上比 Last-Modified 更加准确，优先级也更高。
* 当 Etag 和 Last-Modified 同时存在时，以 Etag 为准。
 
##### 两种缓存方式比较
* 在精确度上，Etag 要优于 Last-Modified，Last-Modified 的时间单位是秒，如果某个文件在 1 秒内改变了多次，那么他们的 Last-Modified 其实并没有体现出来修改，但是 Etag 每次都会改变确保了精度。
* 在性能上，Etag 要逊于 Last-Modified，毕竟 Last-Modified 只需要记录时间，而 Etag 需要服务器通过算法来计算出一个 hash 值。
* 在优先级上，服务器校验优先考虑 Etag。

### 3.3、强制缓存和协商缓存流程
![80097-99c913eb5f04daca.png](https://pic.imgdb.cn/item/618a60bd2ab3f51d91836f4d.png)

### 3.4、私有缓存（浏览器级缓存，默认）
只能用于单独的用户：Cache-Control: Private

### 3.5、共享缓存（代理级缓存）
可以被多个用户使用: Cache-Control: Public

## 五、注意的几点

1. 强缓存情况下，只要缓存还没过期，就会直接从缓存中取数据，就算服务器端有数据变化，也不会从服务器端获取了，这样就无法获取到修改后的数据。决解的办法有：在修改后的资源加上随机数,确保不会从缓存中取。

    例如：
    http://www.kimshare.club/kim/common.css?v=22324432
 
2. 尽量减少304的请求，因为我们知道，协商缓存每次都会与后台服务器进行交互，所以性能上不是很好。如果是静态页面，可以判断文件最近一次的修改时间（Last-Modified），获取文件上次修改时间的消耗比拿到整个数据的消耗要小的多。

## 六、浏览器中如何刷新缓存
* 正常重新加载：F5触发的HTTP请求的请求头中通常包含了If-Modified-Since 或 If-None-Match字段,或者两者兼有。不走强制缓存，可能会走协商缓存。

* 硬性重新加载：chrome中ctrl+shift+R / CTRL+F5触发的HTTP请求的请求头中没有上面的那两个头,却有Pragma: no-cache 或 Cache-Control: no-cache 字段,或者两者兼有。

## 七、服务器配置http缓存
用户设备在Apache上缓存静态内容时，配置如下，配置在项目的.htaccess文件中。
```
<FilesMatch “.(ico|pdf|flv|jpg|jpeg|png|gif|js|css|swf)$”>
Header set Cache-Control “max-age=604800, public”
</FilesMatch>
 
<FilesMatch “.(xml|txt)$”>
Header set Cache-Control “max-age=18000, public, must-revalidate”
</FilesMatch>
 
<FilesMatch “.(html|htm|php)$”>
Header set Cache-Control “max-age=3600, must-revalidate”
```

用户设备在Nginx上缓存静态内容时，配置如下：

```sh
 Location ~ .*\.(jpg|png|css|js)$ {
     Expires 7d;
}
``` 

## 八、简单说下浏览器缓存
![AB4D7237-0A4D-42E8-962D-3C7F29BD5294.png](https://pic.imgdb.cn/item/618a8b852ab3f51d9194d67c.png)
