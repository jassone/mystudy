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
kill  84978073 # 终止事务  好像不行

在 MySQL 中，你不能直接提交其他会话中的事务。事务是由启动它的会话来管理的。如果你在 INFORMATION_SCHEMA.INNODB_TRX 表中看到一个运行中的事务，那么这个事务必须由启动它的会话来提交或回滚。

如果你无法访问启动事务的会话，你可能需要考虑以下选项：

 如果你知道哪个用户启动了事务，你可以尝试联系该用户，让他们提交或回滚事务。
 如果事务是由某个应用程序启动的，你可能需要检查该应用程序，看看它是否卡在某个地方，无法提交或回滚事务。
 如果以上都不可行，你可能需要考虑重启 MySQL 服务器来清除所有未完成的事务。但是请注意，这应该是最后的手段，因为它可能会影响到其他用户和应用程序。
请注意，直接操作数据库，特别是涉及事务的操作，都需要谨慎处理，以防止数据丢失或损坏。
```

