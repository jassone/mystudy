## 查询执行计划

```sql
explain syntax SELECT sku_id from t_stock where total_amount = 1000
查看真正执行的sql -> SELECT sku_id FROM t_stock  PREWHERE total_amount = 1000
```

 

todo



## 相关wiki

- Clickhouse Explain https://blog.csdn.net/SpringBoots/article/details/121104675