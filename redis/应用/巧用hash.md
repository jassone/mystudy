## 巧用hash

## 一、举个例子

假如所存图片3亿，需要解决一个问题：想知道每一张照片的作者是谁（通过图片ID反查用户UID），并且要求查询速度要相当的块，如果把它放到内存中使用String结构做key-value:

```sh
SET media:1155315 939
```

但是这样会非常吃内存，优化方案如下：

将数据分段，每一段使用一个Hash结构存储。

```sh
HSET "mediabucket:1155" "1155315" "939"
```

由于Hash结构会在单个Hash元素在不足一定数量时进行压缩存储，所以可以大量节约内存。这一点在上面的String结构里是不存在的。

- hash-max-zipmap-entries 64  #配置字段最多64个

  含义是当value这个Map内部不超过多少个成员时会采用线性紧凑格式存储，默认是64,即value内部有64个以下的成员就是使用线性紧凑存储，超过该值自动转成真正的HashMap。

- hash-max-zipmap-value 512 #配置value最大为512字节

  含义是当 value这个Map内部的每个成员值长度不超过多少字节就会采用线性紧凑存储来节省空间。

以上2个条件任意一个条件超过设置值都会转换成真正的HashMap，也就不会再节省内存了，那么这个值是不是设置的越大越好呢，答案当然是否定的，HashMap的优势就是查找和操作的时间复杂度都是O(1)的，而放弃Hash采用一维存储则是O(n)的时间复杂度，如果成员数量很少，则影响不大，否则会严重影响性能，所以要权衡好这个值的设置，总体上还是最根本的时间成本和空间成本上的权衡。

而这个一定数量是由配置文件中的hash-max-zipmap-entries参数来控制的。**经过实验，将hash-max-zipmap-entries设置为1000时，性能比较好**。

todo

https://www.bloghome.com.cn/post/redisde-hashlei-xing-zou-kan-kan.html