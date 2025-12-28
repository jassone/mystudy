## main

先要给一个域名开通cdn解析记录(在域名解析后台设置)，然后域名被解析以后，会指向一个cdn网络专用dns服务器 ，然后返回一个ip，这个ip指向一个cdn负载均衡服务器，然后它会找到最近的一台服务器ip，返回给浏览器，浏览器再去请求这台cdn





1. 浏览器发起图片 URL 请求，经过本地 DNS 解析，会将域名解析权交给域名 [CNAME](https://zhida.zhihu.com/search?content_id=210200174&content_type=Article&match_order=1&q=CNAME&zhida_source=entity) 指向的 CDN 专用 DNS 服务器。
2. CDN 的 DNS 服务器将 CDN 的[全局负载均衡设备](https://zhida.zhihu.com/search?content_id=210200174&content_type=Article&match_order=1&q=全局负载均衡设备&zhida_source=entity) IP 地址返回给浏览器。
3. 浏览器向 CDN 全局负载均衡设备发起 URL 请求。
4. CDN 全局负载均衡设备根据用户 IP 地址，以及用户请求的 URL，选择一台用户所属区域的区域负载均衡设备，向其发起请求。
5. 区域负载均衡设备会为用户选择最合适的 CDN 缓存服务器（考虑的依据包括：服务器负载情况，距离用户的距离等），并返回给全局负载均衡设备。
6. 全局负载均衡设备将选中的 CDN 缓存服务器 IP 地址返回给用户。
7. 用户向 CDN 缓存服务器发起请求，缓存服务器响应用户请求，最终将用户所需要偶的内容返回给浏览器。





分发策略：最近，拥堵情况

gslb 全局负载均衡



## 相关技术文档

- https://zhuanlan.zhihu.com/p/549300611