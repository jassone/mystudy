## lua脚本
## 一、基础知识
如果需要扩充若干指令原子性执行时，仅使用原生命令便无法完成。

Redis 为这样的用户场景提供了 lua 脚本支持，用户可以向服务器发送 lua 脚本来执行自定义动作，获取脚本的响应数据。**Redis 服务器会单线程原子性执行 lua 脚本，保证 lua 脚本在处理的过程中不会被任意其它请求打断。**

在2.6版本推出了 lua 脚本功能，允许开发者使用Lua语言编写脚本传到Redis中执行。

## 二、不支持回滚
同redis事务一样，redis使用lua可以保证指令依次执行而不受其他指令干扰，但是不能保证指令最终必定是原子性的。

不能保证原子性的场景主要有以下这些
* 指令语法错误，如上述执行redis.call(‘het’,‘k1’,‘1’),正确的应该是redis.call(‘het’,‘k1’,‘1’,‘2’)。
* 语法是正确的，但是类型不对，比如对已经存在的string类型的key，执行hset等。
* 服务器挂掉了，比如lua脚本执行了一半，但是服务器挂掉了。
 
## 三、相关wiki
* 官方 http://doc.redisfans.com/script/
* Redis中使用Lua脚本（一） https://zhuanlan.zhihu.com/p/77484377
* Redis中使用Lua脚本（二）之红包雨的抢夺 https://zhuanlan.zhihu.com/p/77487460
* Redis中使用Lua脚本（续）- Linux下Lua-cjson开源库的安装和使用 https://zhuanlan.zhihu.com/p/77780939