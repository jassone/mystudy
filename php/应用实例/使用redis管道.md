## 使用Redis管道技术

客户端向服务器发送查询，并从套接字读取，通常以阻塞的方式，用于服务器响应。

服务器处理命令并将响应发送回客户端。如果需要一次执行多个redis命令，以往的方式需要发送多次命令请求，有redis服务器依次执行，并返回结果，

为了解决此类问题，设计者设计出了redis管道命令。

**客户端可以向服务器发送多个请求，而不必等待回复，并最终在一个步骤中读取回复，从而大大增加了协议性能。**

**但是没有任何原子性的保证。**

代码示例：
```php
<?php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$pipe = $redis->multi(Redis::PIPELINE);
for ($i = 0; $i < 3; $i++) {
    $key = "key::{$i}";
    print_r($pipe->set($key, str_pad($i, 2, '0', 0)));
    echo PHP_EOL;
    print_r($pipe->get($key));
    echo PHP_EOL;
}
$result = $pipe->exec();
print_r($result);
```

打印
```
Redis Object
(
)

Redis Object
(
)

Redis Object
(
)

Redis Object
(
)

Redis Object
(
)

Redis Object
(
)

Array
(
    [0] => 1
    [1] => 00
    [2] => 1
    [3] => 01
    [4] => 1
    [5] => 02
)
```
结果如下图，可以看出每次执行set/get命令，并没有被redis服务器立即执行，执行结果被放在了最后的result中。

如果我们像取消管道操作，用下面代码即可：
```php
$pipe->discard();
```