## ORM
ORM 是 Object Relational Mapping 的缩写，译为“对象关系映射”，它解决了对象和关系型数据库之间的数据交互问题。



## 一、关联关系

### 1、Belongs To(一对一)

`belongs to` 会与另一个模型建立了一对一的连接。 这种模型的每一个实例都“属于”另一个模型的一个实例。

```
// `User` 属于 `Company`，`CompanyID` 是外键
type User struct {
  gorm.Model
  Name      string
  CompanyID int
  Company   Company
}

type Company struct {
  ID   int
  Name string
}
```

### 2、has one

`has one` 与另一个模型建立一对一的关联，但它和一对一关系有些许不同。 这种关联表明一个模型的每个实例都包含或拥有另一个模型的一个实例。

```
// User 有一张 CreditCard，UserID 是外键
type User struct {
	gorm.Model
	CreditCard CreditCard // 与CreditCard表有关联关系
}

type CreditCard struct {
	gorm.Model
	Number string
	UserID uint  //关联字段在此表
}
```

### 3、Has Many

`has many` 与另一个模型建立了一对多的连接。 不同于 `has one`，拥有者可以有零或多个关联模型。

```
// User 有多张 CreditCard，UserID 是外键
type User struct {
  gorm.Model
  CreditCards []CreditCard
}

type CreditCard struct {
  gorm.Model
  Number string
  UserID uint
}
```

### 4、Many To Many

Many to Many 会在两个 model 中添加一张连接表。

```
// User 拥有并属于多种 language，`user_languages` 是连接表
type User struct {
  gorm.Model
  Languages []Language `gorm:"many2many:user_languages;"`
}

type Language struct {
  gorm.Model
  Name string
}
```

### 5、多态

todo





## 相关wiki

-  ORM是什么 https://www.ruanyifeng.com/blog/2019/02/orm-tutorial.html
-  关联关系 https://www.cnblogs.com/liuqingzheng/p/16244561.html