## LRU算法简单实现
原理:使用数组队列，将最近使用的key优先放到队列头部，如果队列长度超过设置的长度，则删除队列尾部的key。

```php
<?php
/**
 * LRU是最近最少使用页面置换算法(Least Recently Used),也就是首先淘汰最长时间未被使用的页面
 */
class LRU_Cache
{

    private $array_lru = array();
    private $max_size = 0;

    function __construct($size)
    {
        // 缓存最大存储
        $this->max_size = $size;
    }

    public function set_value($key, $value)
    {
        // 如果存在，则向队尾移动，先删除，后追加
        if (array_key_exists($key, $this->array_lru)) {
            unset($this->array_lru[$key]);
        }
        // 长度检查，超长则删除首元素
        if (count($this->array_lru) >= $this->max_size) {
            array_shift($this->array_lru);
        }
        // 队尾追加元素
        $this->array_lru[$key] = $value;
    }

    public function get_value($key)
    {
        $ret_value = false;

        if (array_key_exists($key, $this->array_lru)) {
            $ret_value = $this->array_lru[$key];
            // 移动到队尾
            unset($this->array_lru[$key]);
            $this->array_lru[$key] = $ret_value;
        }
        
        return $ret_value;
    }

    public function vardump_cache()
    {
        var_dump($this->array_lru);
    }
}

$cache = new LRU_Cache(5);
$cache->set_value("01", "01");
$cache->set_value("02", "02");
$cache->set_value("03", "03");
$cache->set_value("04", "04");
$cache->set_value("05", "05");
$cache->vardump_cache();
$cache->set_value("06", "06");
$cache->set_value("03", "03");
$cache->set_value("07", "07");
$cache->set_value("01", "01");
$cache->vardump_cache();
```

```
// 最后一个输出
array(5) {
  ["05"]=>
  string(2) "05"
  ["06"]=>
  string(2) "06"
  ["03"]=>
  string(2) "03"
  ["07"]=>
  string(2) "07"
  ["01"]=>
  string(2) "01"
}
```

其他实现TODO
https://blog.csdn.net/youyangyouni/article/details/80988852

https://github.com/rogeriopvl/php-lrucache