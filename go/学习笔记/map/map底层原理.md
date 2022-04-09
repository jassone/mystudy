## map底层原理
todo

hash冲突解决
* 开放地址法
* 拉链法

hash表扩容
负载因子 (6.5 )   15限制
需要重新迁移所有hash数据到新的空间，每次操作hash表，迁移一点点，这种方式叫做：渐进式扩容，避免一次性扩容带来的性能抖动。

溢出桶

扩容规则

等量扩容：  让key value排列更加紧凑(因为某些key被删除导致不紧凑)



## wiki
* https://www.topgoer.com/go基础/Map实现原理.html