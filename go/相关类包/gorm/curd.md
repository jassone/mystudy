## curd

## 一、增

- Create
- CreateInBatches

[Upsert](https://gorm.io/zh_CN/docs/create.html#upsert) 和 [Create With Associations](https://gorm.io/zh_CN/docs/create.html#create_with_associations)同样支持批量插入 ??

批量插入，才返回数量？？待确认





## 二、查

first 换成 take

```
// 获取第一条记录（主键升序）
db.First(&user)

// 获取一条记录，没有指定排序字段
db.Take(&user)

// 获取最后一条记录（主键降序）
db.Last(&user)

如果你想避免ErrRecordNotFound错误，你可以使用Find，比如db.Limit(1).Find(&user)，Find方法可以接受struct和slice的数据。

// 只能查主键
db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);


指定结构体查询字段
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;

选择特定字段
Select allows you to specify the fields that you want to retrieve from database. Otherwise, GORM will select all fields by default.

db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;



######Joins 预加载
db.Joins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id`;

// inner join
db.InnerJoins("Company").Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` INNER JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id`;
Join with conditions

db.Joins("Company", db.Where(&Company{Alive: true})).Find(&users)
// SELECT `users`.`id`,`users`.`name`,`users`.`age`,`Company`.`id` AS `Company__id`,`Company`.`name` AS `Company__name` FROM `users` LEFT JOIN `companies` AS `Company` ON `users`.`company_id` = `Company`.`id` AND `Company`.`alive` = true;


##### 高级查询

// 智能选择字段
type APIUser struct {
  ID   uint
  Name2222 string // 转换
}

// 在查询时，GORM 会自动选择 `id `, `name` 字段
db.Model(&User{}).Limit(10).Find(&APIUser{})
// SQL: SELECT `id`, `Name2222` FROM `users` LIMIT 10
//  QueryFields 模式中, 所有的模型字段（model fields）都会被根据他们的名字选择。



Pluck
GORM 中的 Pluck 方法用于从数据库中查询单列并扫描结果到片段（slice）。 当您需要从模型中检索特定字段时，此方法非常理想。

如果需要查询多个列，可以使用 Select 配合 Scan 或者 Find 来代替。

// 检索所有用户的 age
var ages []int64
db.Model(&User{}).Pluck("age", &ages)

// 检索所有用户的 name
var names []string
db.Model(&User{}).Pluck("name", &names)

// 从不同的表中检索 name
db.Table("deleted_users").Pluck("name", &names)

// 使用Distinct和Pluck
db.Model(&User{}).Distinct().Pluck("Name", &names)
// SQL: SELECT DISTINCT `name` FROM `users`

// 多列查询
db.Select("name", "age").Scan(&users)
db.Select("name", "age").Find(&users)


### preload
条件预加载
GORM allows Preload associations with conditions, it works similar to Inline Conditions

// Preload Orders with conditions
db.Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
// SELECT * FROM users;
// SELECT * FROM orders WHERE user_id IN (1,2,3,4) AND state NOT IN ('cancelled');

db.Where("state = ?", "active").Preload("Orders", "state NOT IN (?)", "cancelled").Find(&users)
// SELECT * FROM users WHERE state = 'active';
// SELECT * FROM orders WHERE user_id IN (1,2) AND state NOT IN ('cancelled');



### Example of Safe Reuse
To safely reuse a *gorm.DB instance, use a New Session Method:

queryDB := DB.Where("name = ?", "jinzhu").Session(&gorm.Session{})

// First query
queryDB.Where("age > ?", 10).First(&user)
// SQL: SELECT * FROM users WHERE name = "jinzhu" AND age > 10

// Second query, safely isolated
queryDB.Where("age > ?", 20).First(&user2)
// SQL: SELECT * FROM users WHERE name = "jinzhu" AND age > 20
```



## 更新

```
NOTE不要将 Save 和 Model一同使用, 这是 未定义的行为。


批量更新
// Update with struct
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin';

// Update with map
db.Table("users").Where("id IN ?", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);


返回修改行的数据
// return all columns
var users []User
db.Model(&users).Clauses(clause.Returning{}).Where("role = ?", "admin").Update("salary", gorm.Expr("salary * ?", 2))
// UPDATE `users` SET `salary`=salary * 2,`updated_at`="2021-10-28 17:37:23.19" WHERE role = "admin" RETURNING *
// users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}

// return specified columns
db.Model(&users).Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Update("salary", gorm.Expr("salary * ?", 2))
// UPDATE `users` SET `salary`=salary * 2,`updated_at`="2021-10-28 17:37:23.19" WHERE role = "admin" RETURNING `name`, `salary`
// users => []User{{ID: 0, Name: "jinzhu", Role: "", Salary: 100}, {ID: 0, Name: "jinzhu.2", Role: "", Salary: 1000}}

```



## 删除





## 其他



- GORM 默认会将单个的 create, update, delete 操作**封装在事务内**进行处理，

  对于写操作（创建、更新、删除），为了确保数据的完整性，GORM 会将它们封装在一个事务里。但这会降低性能，你可以在初始化时禁用这种方式

  ```
  db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
    SkipDefaultTransaction: true,
  })
  ```





## json的使用

	// 格式化数据
	requestByte, _ := json.Marshal(request)
	auditParam := datatypes.JSON{}
	auditParam.Scan(requestByte)
	
	// 更新数据库
	schoolMinaCodeUpdate := &schoolMinaCodeModel.LearningSchoolMinaCode{
		AuditID:       uint64(auditId),
		AuditParam:    auditParam, // 定义为 datatypes.JSON 
	}

