---
title: gorm的使用
toc_number: true
copyright: false
date: 2020-03-02 19:29:51
categories: [Go学习笔记]
tags: [golang]
---

原文地址：[Go orm框架gorm学习](https://www.cnblogs.com/rickiyang/p/11074162.html)

之前学习过原生的Go连接MYSQL的方法，使用Go自带的`"database/sql"`数据库连接api，`"github.com/go-sql-driver/mysql"`MYSQL驱动，通过比较原生的写法去写sql和处理事务。目前开源界也有很多封装好的orm操作框架，帮我们简省一些重复的操作，提高代码可读性。`gorm`就是这样的一款作品，我们来学习一下gorm的操作流程。

## 安装

```
go get -u github.com/jinzhu/gorm
```

## 数据库连接

要连接到数据库首先要导入驱动程序。例如

```
import _ "github.com/go-sql-driver/mysql"
```

为了方便记住导入路径，GORM包装了一些驱动。

```
import _ "github.com/jinzhu/gorm/dialects/mysql"
// import _ "github.com/jinzhu/gorm/dialects/postgres"
// import _ "github.com/jinzhu/gorm/dialects/sqlite"
// import _ "github.com/jinzhu/gorm/dialects/mssql"
```

所以包名可以改为如上：

```go
import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
    db, err := gorm.Open("mysql", "username:password@tcp(IP:port)/dbname?charset=utf8&parseTime=True&loc=Local")
  	db.DB().SetMaxIdleConns(10)
	db.DB().SetMaxOpenConns(100)
  	defer db.Close()
}
```

## 数据模型定义

### 表名，列名如何对应结构体

在Gorm中，默认规则为：表名是结构体名的复数形式，列名是字段名的蛇形小写。

**即，如果有一个user表，那么如果你定义的结构体名为：User，gorm会默认表名为users而不是user。**

例如有如下表结构定义：

```sql
CREATE TABLE `areas` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `area_id` int(11) NOT NULL COMMENT '区县id',
  `area_name` varchar(45) NOT NULL COMMENT '区县名',
  `city_id` int(11) NOT NULL COMMENT '城市id',
  `city_name` varchar(45) NOT NULL COMMENT '城市名称',
  `province_id` int(11) NOT NULL COMMENT '省份id',
  `province_name` varchar(45) NOT NULL COMMENT '省份名称',
  `area_status` tinyint(3) NOT NULL DEFAULT '1' COMMENT '该条区域信息是否可用 ： 1:可用  2：不可用',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='区域表'
```

那么对应的结构体定义如下：

```go
type Area struct {
	Id int
	AreaId int
	AreaName string
	CityId int
	CityName string
	ProvinceId int
	ProvinceName string
	AreaStatus int
	CreatedAt time.Time
	UpdatedAt time.Time
}
```

如何全局禁用表名复数呢？

可以在创建数据库连接的时候设置如下参数：

```go
// 全局禁用表名复数
db.SingularTable(true) // 如果设置为true,`User`的默认表名为`user`,使用`TableName`设置的表名不受影响
```

这样的话，表名默认即为结构体的首字母小写形式。

## CRUD 使用

下面我们使用一张User表来就CRUD做一些操作示例：

表结构如下：

```sql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) NOT NULL DEFAULT '',
  `age` int(3) NOT NULL DEFAULT '0',
  `sex` tinyint(3) NOT NULL DEFAULT '0',
  `phone` varchar(40) NOT NULL DEFAULT '',
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4
```

首先初始化数据库连接：

```go
package main

import (
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

var db *gorm.DB

type User struct {
	Id int
	Name string
	Age int
	Sex byte
	Phone string
}

func init() {
	var err error
	db, err = gorm.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test?charset=utf8&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}
	//设置全局表名禁用复数
	db.SingularTable(true)
}
```

下面所有的操作都是在上面的初始化连接上执行的操作。

### 插入

```go
//插入数据
func (user *User) Insert()  {
	//这里使用了Table()函数，如果你没有指定全局表名禁用复数，或者是表名跟结构体名不一样的时候
	//你可以自己在sql中指定表名。这里是示例，本例中这个函数可以去除。
	db.Table("user").Create(user)
}
```

### 更新

```go
//注意，Model方法必须要和Update方法一起使用
//使用效果相当于Model中设置更新的主键key（如果没有where指定，那么默认更新的key为id），Update中设置更新的值
//如果Model中没有指定id值，且也没有指定where条件，那么将更新全表
//相当于：update user set name='xiaoming' where id=1;
user := User{Id: 1,Name:"xiaoming"}
db.Model(&user).Update(user)

//注意到上面Update中使用了一个Struct，你也可以使用map对象。
//需要注意的是：使用Struct的时候，只会更新Struct中这些非空的字段。
//对于string类型字段的""，int类型字段0，bool类型字段的false都被认为是空白值，不会去更新表

//下面这个更新操作只使用了where条件没有在Model中指定id
//update user set name='xiaohong' wehre sex=1
db.Model(&User{}).Where("sex = ?",1).Update("name","xiaohong")
```

如果你想手动将某个字段set为空值, 可以使用单独选定某些字段的方式来更新：

```go
user := User{Id: 1}
db.Model(&user).Select("name").Update(map[string]interface{}{"name":"","age":0})
```

忽略掉某些字段：

当你的更新的参数为结构体，而结构体中某些字段你又不想去更新，那么可以使用Omit方法过滤掉这些不想update到库的字段：

```go
user := User{Id: 1,Name:"xioaming",Age:12}
db.Model(&user).Omit("name").Update(&user)
```

### 删除

```go
//delete from user where id=1;
user := User{Id: 1}
db.Delete(&user)

//delete from user where id > 11;
db.Delete(&User{},"id > ?",11)
```

### 事务

```go
func CreateAnimals(db *gorm.DB) err {
  tx := db.Begin()
  // 注意，一旦你在一个事务中，使用tx作为数据库句柄

  if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
     tx.Rollback()
     return err
  }

  if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
     tx.Rollback()
     return err
  }

  tx.Commit()
  return nil
}
```

### 查询

```go
func (user *User) query() (u []User) {
	//查询所有记录
	db.Find(&u)

	//Find方法可以带 where 参数
	db.Find(&u,"id > ? and age > ?",2,12)

	//带where 子句的查询，注意where要在find前面
	db.Where("id > ?", 2).Find(&u)

	// where name in ("xiaoming","xiaohong")
	db.Where("name in (?)",[]string{"xiaoming","xiaohong"}).Find(&u)

	//获取第一条记录，按照主键顺序排序
	db.First(&u)

	//First方法可以带where 条件
	db.First(&u,"where sex = ?",1)

	//获取最后一条记录，按照主键顺序排序
	//同样 last方法也可以带where条件
	db.Last(&u)

	return u
}
```

注意：方法中带的`&u`表示是返回值用u这个对象来接收。

上面的查询都将返回表中所有的字段，如果你想指定查询某些字段该怎么做呢？

### 指定查询字段-Select

```go
//指定查询字段
db.Select("name,age").Where(map[string]interface{}{"age":12,"sex":1}).Find(&u)
```

### 使用Struct和map作为查询条件

```go
//使用Struct，相当于：select * from user where age =12 and sex=1
db.Where(&User{Age:12,Sex:1}).Find(&u)

//等同上一句
db.Where(map[string]interface{}{"age":12,"sex":1}).Find(&u)
```

### not 条件的使用

```go
//where name not in ("xiaoming","xiaohong")
db.Not("name","xiaoming","xiaohong").Find(&u)

//同上
db.Not("name",[]string{"xiaoming","xiaohong"}).Find(&u)
```

### or 的使用

```go
//where age > 12 or sex = 1
db.Where("age > ?",12).Or("sex = ?",1).Find(&u)
```

### order by 的使用

```go
//order by age desc
db.Where("age > ?",12).Or("sex = ?",1).Order("age desc").Find(&u)
```

### limit 的使用

```go
//limit 10
db.Not("name",[]string{"xiaoming","xiaohong"}).Limit(10).Find(&u)
```

### offset 的使用

```go
//limit 300,10
db.Not("name",[]string{"xiaoming","xiaohong"}).Limit(10).Offset(300).Find(&u)
```

### count(*)

```go
//count(*)
var count int
db.Table("user").Where("age > ?",0).Count(&count)
```

注意：这里你在指定表名的情况下sql为：select count(*) from user where age > 0;

如上代码如果改为：

```go
var count int
var user []User
db.Where("age > ?",0).Find(&user).Count(&count)
```

相当于你先查出来[]User，然后统计这个list的长度。跟你预期的sql不相符。

### group & having

```go
rows, _ := db.Table("user").Select("count(*),sex").Group("sex").
		Having("age > ?", 10).Rows()
for rows.Next() {
    fmt.Print(rows.Columns())
}
```

### join

```go
db.Table("user u").Select("u.name,u.age").Joins("left join user_ext ue on u.user_id = ue.user_id").Row()
```

如果有多个连接，用多个Join方法即可。

### 原生函数

```go
db.Exec("DROP TABLE user;")
db.Exec("UPDATE user SET name=? WHERE id IN (?)", "xiaoming", []int{11,22,33})
db.Exec("select * from user where id > ?",10).Scan(&user)
```

### 一些函数

**FirstOrInit 和 FirstOrCreate**

获取第一个匹配的记录，若没有，则根据条件初始化一个新的记录：

```
//注意：where条件只能使用Struct或者map。如果这条记录不存在，那么会新增一条name=xiaoming的记录
db.FirstOrInit(&u,User{Name:"xiaoming"})
//同上
db.FirstOrCreate(&u,User{Name:"xiaoming"})
```

**Attrs**

如果没有找到记录，则使用Attrs中的数据来初始化一条记录：

```go
//使用attrs来初始化参数，如果未找到数据则使用attrs中的数据来初始化一条
//注意：attrs 必须 要和FirstOrInit 或者 FirstOrCreate 连用
db.Where(User{Name:"xiaoming"}).Attrs(User{Name:"xiaoming",Age:12}).FirstOrInit(&u)
```

**Assign**

```go
//不管是否找的到，最终返回结构中都将带上Assign指定的参数
db.Where("age > 12").Assign(User{Name:"xiaoming"}).FirstOrInit(&u)
```

**Pluck**

如果user表中你只想查询age这一列，该怎么返回呢，gorm提供了Pluck函数用于查询单列，返回数组：

```go
var ages []int
db.Find(&u).Pluck("age",&ages)
```

**Scan**

Scan函数可以将结果转存储到另一个结构体中。

```go
type SubUser struct{
    Name string
    Age int
}

db.Table("user").Select("name,age").Scan(&SubUser)
```

**sql.Row & sql.Rows**

row和rows用户获取查询结果。

```go
//查询一行
row := db.Table("user").Where("name = ?", "xiaoming").Select("name, age").Row() // (*sql.Row)
//获取一行的结果后，调用Scan方法来将返回结果赋值给对象或者结构体
row.Scan(&name, &age)

//查询多行
rows, err := db.Model(&User{}).Where("sex = ?",1).Select("name, age, phone").Rows() // (*sql.Rows, error)
defer rows.Close()
for rows.Next() {
    ...
    rows.Scan(&name, &age, &email)
    ...
}
```

## 日志

Gorm有内置的日志记录器支持，默认情况下，它会打印发生的错误。

```go
// 启用Logger，显示详细日志
db.LogMode(true)

// 禁用日志记录器，不显示任何日志
db.LogMode(false)

// 调试单个操作，显示此操作的详细日志
db.Debug().Where("name = ?", "xiaoming").First(&User{})
```