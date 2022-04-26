## 使用redis lua脚本

```php
$lua = <<<LUA
    local sequenceKey = KEYS[1]
    local time = KEYS[2]
    local sequenceNumber = redis.call("incr", sequenceKey)
    redis.call("expire", sequenceKey, time)
    return sequenceNumber
LUA;

$redis = new Redis();
$redis->connect('127.0.0.1');
$redis->select(0);
$sequence = $redis->evalsha(sha1($lua), ['lua1234', 120], 2);
$luaError = $redis->getLastError();
if (isset($luaError)) {
    print_r($luaError);
}
print_r($sequence);
//sequence = 1 2 3
```
