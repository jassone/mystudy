
## CLI SAPI下 getopt参数处理函数
getopt()函数是PHP自带专门用来从命令行参数列表中获取选项。原型如下：

```
getopt(string $short_options, array $long_options = [], int &$rest_index = null): array|false
```

##### 示例和用法
```
arg_test.php 代码
$options = getopt('a:b:cde');
var_dump($options);

以下面的命令启动：
php arg_test.php -aav -c -d -b bv -f
则输出为：
array(4) {
  ["a"]=>
  string(2) "av"
  ["c"]=>
  bool(false)
  ["d"]=>
  bool(false)
  ["b"]=>
  string(2) "bv"
}
```
 
-------

### 未完，其他详见官网wiki

**https://www.php.net/manual/zh/function.getopt.php**