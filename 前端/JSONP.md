## jsonp

## 一、JSONP的由来

**JSONP是JSON with Padding的略称。**

根据浏览器同源策略，所谓同源就是协议、主机、端口号都相同时成为同源。a 域的js不能直接访问 b域名的信息。

**但是script 标签的src属性可以跨域引用文件，jsonp是请求之后后台包装好一段json，并且把数据放在一个callback函数，返回一个js文件，动态引入这个文件，下载完成js之后，会去调用这个callback,通过这样访问数据。**

但是只支持get方式请求。

## 二、JSONP有什么用

由于同源从略的限制，XMLHttpRequest只允许请求前源（域名、协议、端口）的资源，为了实现跨域请求，可以通过script标签实现跨域请求，然后再服务端输出JSON数据并执行回调函数，从而解决跨域数据请求

## 三、如何使用JSONP

 HTML代码

![ererePG.jpeg](https://pic.imgdb.cn/item/62d409daf54cd3f937c20097.jpg)

 

## 四、JSONP原理

首先在客户端注册一个callback，然后把callback的名字传给服务器。此时，服务器先生成json数据，然后以javascript语法的方式，生成function，function名字就是传递上来的带参数jsonp。最后将json数据直接以入参的方式，放置function中，这样就生成js语法的文档，返回给客户端。客户端浏览器，解析script变迁，并执行返回javascript文档，此时数据作为参数，传入了客户端预先定义好的callback函数里。简单的说，就是利用script标签没有跨域限制的“漏洞”来达到与第三方通讯的目的。

![sdsfdsfdsf.jpeg](https://pic.imgdb.cn/item/62d40a26f54cd3f937c2686e.jpg)

总结一下，json 是一种数据格式，jsonp 是一种数据调用的方式，带callback的json就是jsonp。