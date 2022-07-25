## OpenResty

## 一、OpenResty 介绍

OpenResty(又称：ngx_openresty) 是一个基于 NGINX 的可伸缩的 Web 平台，提供了很多高质量的第三方模块。

OpenResty 是一个强大的 Web 应用服务器，Web 开发人员可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块,更主要的是在性能方面，OpenResty可以 快速构造出足以胜任 单机10K 以上并发连接响应的超高性能 Web 应用系统。

OpenResty 的目标是让你的 Web 服务直接跑在 Nginx 服务内部,充分利用 Nginx 的非阻塞 I/O 模型,不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL,PostgreSQL,~Memcaches 以及 ~Redis 等都进行一致的高性能响应。

所以对于一些高性能的服务来说，**可以直接使用 OpenResty 访问 Mysql或Redis等，而不需要通过第三方语言（PHP、Python、Ruby）等来访问数据库再返回，这大大提高了应用的性能。**



## 相关wiki

- OpenResty 中文官网：http://openresty.org/cn/

