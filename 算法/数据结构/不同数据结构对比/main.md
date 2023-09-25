## main

### 1、平衡二叉树和跳表比较

实现有序数据结构典型实现有平衡数和跳表

跳表优点：跳表的插入、删除和查找操作的平均[时间复杂度](https://www.zhihu.com/search?q=时间复杂度&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A3139987854})都是O(log n)

Redis使用跳表而不是平衡树的原因

- 跳表的实现比树简单，不需要考虑平衡问题。
- 跳表的插入和删除操作比树快，只需要修改相邻节点的指针。
- 跳表可以方便地实现范围查询和排名查询，而树需要额外的遍历操作(二叉树在多次插入删除后，需要rebalance来重新调整结果平衡)。
- 跳表占用的内存空间比树小，因为跳表的节点数目和层数都是随机生成的。

* 维护结构平衡的成本比较低，完全依靠随机(而二叉树在多次插入删除后，需要rebalance来重新调整结果平衡)

