### 设置超时时间

```
$ctx = stream_context_create(array(
  'http' => array(
       'timeout' => 10    //设置一个超时时间，单位为秒
  )
));

file_get_contents($str, 0, $ctx);
```
 