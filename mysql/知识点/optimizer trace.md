## optimizer trace
设计 MySQL 的大叔贴心的为这部分小伙伴提出了一个 optimizer trace 的功能，这个功能可以让我们方便的查看优化器生成执行计划的整个过程。

```sql
SET optimizer_trace="enabled=on";

SELECT * FROM s1 WHERE 
 key1 > 'z' AND 
 key2 < 1000000 AND 
 key3 IN ('a', 'b', 'c') AND 
 common_field = 'abc';
 
SELECT * FROM information_schema.OPTIMIZER_TRACE\G
```

...TODO

