## 系统相关

查询系统配置

```sql
SHOW [GLOBAL|SESSION] VARIABLES [LIKE 匹配的模式];
```

设置系统配置

```sql
SET [GLOBAL|SESSION] 系统变量名 = 值;
或者 
SET [@@(GLOBAL|SESSION).]var_name = XXX;
```

查看状态

```sql
SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

查看字符集

```sql
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
```

查看比较规则

```sql
SHOW COLLATION [LIKE 匹配的模式];
```

事务相关

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX WHERE trx_state = 'RUNNING';
COMMIT 84978073 # 终止事务
```

