## 相关sql语句

### insert ignore into
表示如果中已经存在相同的记录，则忽略当前新数据。

当插入数据时，如出现错误时，如重复数据，将不返回错误，只以警告形式返回。所以使用ignore请确保语句本身没有问题，否则也会被忽略掉。例如：

```sql
INSERT IGNORE INTO books (name) VALUES ('MySQL Manual')
```

### on duplicate key update：

当primary或者unique重复时，则执行update语句，如update后为无用语句，如id=id，则同1功能相同，但错误不会被忽略掉。例如，为了实现name重复的数据插入不报错，可使用一下语句：

```sql
INSERT INTO books (name) VALUES ('MySQL Manual') ON duplicate KEY UPDATE id = id
```
