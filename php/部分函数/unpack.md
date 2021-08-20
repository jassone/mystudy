

```
$data = "a";
 
$bitmap = unpack('C*', $data);

$count = 0;
foreach($bitmap as $key => $number) {
    //主要看这一段哈，更简洁，更有通用性
    while($number) {
        $count += $number & 1;
        $number >>= 1;
    }
}


// 优化
$bitmap = unpack('C*', $data);
 
$count = 0;
foreach($bitmap as $key => $number) {
    //原理是什么？且听我下面慢慢道来
    while($number) {
        $number &= ($number - 1);
        $count++;
    }
}
//echo $count;
```