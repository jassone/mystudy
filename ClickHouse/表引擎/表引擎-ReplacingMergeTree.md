## 表引擎-ReplacingMergeTree

该引擎和MergeTree的不同之处在于它会删除**排序键值**相同的重复项。适用于在后台清除重复的数据以节省空间。

```sql
create table t_stock_replacingmergetree(
  id UInt32,
  sku_id String,
  total_amount Decimal(16,2),
	create_time DateTime
) engine = ReplacingMergeTree(create_time)
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id);

insert into t_stock_replacingmergetree values
(101,'sku_001',10.00,'2020-06-01 12:00:00'),
(102,'sku_002',10.00,'2020-06-01 11:00:00'),
(102,'sku_004',10.00,'2020-06-01 12:00:00'),
(102,'sku_002',10.00,'2020-06-01 13:00:00'),
(102,'sku_002',10.00,'2020-06-01 13:00:00'),
(102,'sku_002',10.00,'2020-06-02 12:00:00');

##查询结果
┌──id─┬─sku_id──┬─total_amount─┬─────────create_time─┐
│ 102 │ sku_002 │           10 │ 2020-06-02 12:00:00 │
└─────┴─────────┴──────────────┴─────────────────────┘
┌──id─┬─sku_id──┬─total_amount─┬─────────create_time─┐
│ 101 │ sku_001 │           10 │ 2020-06-01 12:00:00 │
│ 102 │ sku_002 │           10 │ 2020-06-01 13:00:00 │
│ 102 │ sku_004 │           10 │ 2020-06-01 12:00:00 │
└─────┴─────────┴──────────────┴─────────────────────┘
如果上面sql再执行一次，就会有重复，这时候需要等合并操作后才能去重。
```

 ### 注意点

- 并不能始终保证数据是完全去重的，数据去重只会发生在**同一批插入数据**和**后台数据合并**这两个时机。
- 去重只限定在一个分区内，不能跨区
- 如果有保留版本字段，则取最大的那条；如果没有版本字段或版本字段也相同，则取最后插入的那条。