---
title: Go连接MySQL
date: 2020-03-02 19:14:23
categories: [Go学习笔记]
tags: [golang]
---

原文地址：[Go连接MYSQL](https://www.cnblogs.com/rickiyang/p/11074180.html)

Go原生提供了连接数据库操作的支持，在用 Golang进行开发的时候，如果需要在和数据库交互，则可以使用database/sql包。这是一个对关系型数据库的通用抽象，它提供了标准的、轻量的、面向行的接口。

在Go中访问数据库需要用到`sql.DB`接口：它可以创建语句(statement)和事务(transaction)，执行查询，获取结果。

使用数据库时，除了`database/sql`包本身，还需要引入想使用的特定数据库驱动。官方不提供实现，先下载第三方的实现，[点击这里](https://github.com/golang/go/wiki/SQLDrivers)查看各种各样的实现版本。

本文测试数据库为mysql，使用的驱动为:`github.com/go-sql-driver/mysql`,需要引入的包为：

```go
"database/sql"
_ "github.com/go-sql-driver/mysql"
```

**解释一下导入包名前面的"_"作用：**

*import 下划线（如：import _ github/demo）的作用：当导入一个包时，该包下的文件里所有init()函数都会被执行，然而，有些时候我们并不需要把整个包都导入进来，仅仅是是希望它执行init()函数而已。这个时候就可以使用 import _ 引用该包。*

上面的mysql驱动中引入的就是mysql包中各个init()方法，你无法通过包名来调用包中的其他函数。导入时，驱动的初始化函数会调用sql.Register将自己注册在database/sql包的全局变量sql.drivers中，以便以后通过sql.Open访问。

执行数据库操作之前我们准备一张表：

```sql
CREATE TABLE `user` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT,
    `name` varchar(45) DEFAULT '',
    `age` int(11) NOT NULL DEFAULT '0',
    `sex` tinyint(3) NOT NULL DEFAULT '0',
    `phone` varchar(45) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

## 1. 初始化数据库连接：

```go
DB, _ := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test?charset=utf8&parseTime=True&loc=Local")
//设置数据库最大连接数
DB.SetConnMaxLifetime(100)
//设置上数据库最大闲置连接数
DB.SetMaxIdleConns(10)
//验证连接
if err := DB.Ping(); err != nil {
    fmt.Println("open database fail")
    return
}
fmt.Println("connnect success")
```

sql.Open()中的数据库连接串格式为：`"用户名:密码@tcp(IP:端口)/数据库?charset=utf8&parseTime=True&loc=Local"`。

DB的类型为:`*sql.DB`，有了DB之后我们就可以执行CRUD操作。Go将数据库操作分为两类：`Query`与`Exec`。两者的区别在于前者会返回结果，而后者不会。

- `Query`表示查询，它会从数据库获取查询结果（一系列行，可能为空）。
- `Exec`表示执行语句，它不会返回行。

此外还有两种常见的数据库操作模式：

- `QueryRow`表示只返回一行的查询，作为`Query`的一个常见特例。
- `Prepare`表示准备一个需要多次使用的语句，供后续执行用。

## 2. 查询操作

```go
var user User
rows, e := DB.Query("select * from user where id in (1,2,3)")
if e != nil {
    errors.New("query incur error")
}
for rows.Next(){
    e := rows.Scan(user.sex, user.phone, user.name, user.id, user.age)
    if e != nil{
        fmt.Println(json.Marshal(user))
    }
}
rows.Close()
//单行查询操作
DB.QueryRow("select * from user where id=1").Scan(user.age, user.id, user.name, user.phone, user.sex)
```

整体工作流程如下：

1. 使用`db.Query()`来发送查询到数据库，获取结果集`Rows`，并检查错误。
2. 使用`rows.Next()`作为循环条件，迭代读取结果集。
3. 使用`rows.Scan`从结果集中获取一行结果。
4. 使用`rows.Err()`在退出迭代后检查错误。
5. 使用`rows.Close()`关闭结果集，释放连接。

## 3. 增删改和Exec

通常不会约束你查询必须用Query，只是Query会返回结果集，而Exec不会返回。所以如果你执行的是增删改操作一般用Exec会好一些。Exec返回的结果是`Result`，`Result`接口允许获取执行结果的元数据:

```go
type Result interface {
    // 用于返回自增ID，并不是所有的关系型数据库都有这个功能。
    LastInsertId() (int64, error)
    // 返回受影响的行数。
    RowsAffected() (int64, error)
}
```

## 4. 准备查询

如果你现在想使用占位符的功能，where 的条件想以参数的形式传入，Go提供了`db.Prepare`语句来帮你绑定。准备查询的结果是一个准备好的语句（prepared statement），语句中可以包含执行时所需参数的占位符（即绑定值）。准备查询比拼字符串的方式好很多，它可以转义参数，避免SQL注入。同时，准备查询对于一些数据库也省去了解析和生成执行计划的开销，有利于性能。

### 占位符

PostgreSQL使用`$N`作为占位符，`N`是一个从1开始递增的整数，代表参数的位置，方便参数的重复使用。MySQL使用`?`作为占位符，SQLite两种占位符都可以，而Oracle则使用`:param1`的形式。

```
MySQL               PostgreSQL            Oracle
=====               ==========            ======
WHERE col = ?       WHERE col = $1        WHERE col = :col
VALUES(?, ?, ?)     VALUES($1, $2, $3)    VALUES(:val1, :val2, :val3)
stmt, e := DB.Prepare("select * from user where id=?")
query, e := stmt.Query(1)
query.Scan()
```

## 5. 事务的使用

通过`db.Begin()`来开启一个事务，`Begin`方法会返回一个事务对象`Tx`。在结果变量`Tx`上调用`Commit()`或者`Rollback()`方法会提交或回滚变更，并关闭事务。在底层，`Tx`会从连接池中获得一个连接并在事务过程中保持对它的独占。事务对象`Tx`上的方法与数据库对象`sql.DB`的方法一一对应，例如`Query,Exec`等。事务对象也可以准备(prepare)查询，由事务创建的准备语句会显式绑定到创建它的事务。

```go
//开启事务
tx, err := DB.Begin()
if err != nil {
    fmt.Println("tx fail")
}
//准备sql语句
stmt, err := tx.Prepare("DELETE FROM user WHERE id = ?")
if err != nil {
    fmt.Println("Prepare fail")
    return false
}
//设置参数以及执行sql语句
res, err := stmt.Exec(user.id)
if err != nil {
    fmt.Println("Exec fail")
    return false
}
//提交事务
tx.Commit()
```

我们来一个完整的sql操作：

```go
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
    "github.com/pkg/errors"
    "strings"
)

//数据库配置
const (
    userName = "root"
    password = "123456"
    ip       = "127.0.0.1"
    port     = "3306"
    dbName   = "test"
)

//Db数据库连接池
var DB *sql.DB

type User struct {
    id    int64
    name  string
    age   int8
    sex   int8
    phone string
}

//注意方法名大写，就是public
func InitDB() {
    //构建连接："用户名:密码@tcp(IP:端口)/数据库?charset=utf8"
    path := strings.Join([]string{userName, ":", password, "@tcp(", ip, ":", port, ")/", dbName, "?charset=utf8"}, "")
    //打开数据库,前者是驱动名，所以要导入： _ "github.com/go-sql-driver/mysql"
    DB, _ = sql.Open("mysql", path)
    //设置数据库最大连接数
    DB.SetConnMaxLifetime(100)
    //设置上数据库最大闲置连接数
    DB.SetMaxIdleConns(10)
    //验证连接
    if err := DB.Ping(); err != nil {
        fmt.Println("open database fail")
        return
    }
    fmt.Println("connnect success")
}

//查询操作
func Query() {
    var user User
    rows, e := DB.Query("select * from user where id in (1,2,3)")
    if e == nil {
        errors.New("query incur error")
    }
    for rows.Next() {
        e := rows.Scan(user.sex, user.phone, user.name, user.id, user.age)
        if e != nil {
            fmt.Println(json.Marshal(user))
        }
    }
    rows.Close()
    DB.QueryRow("select * from user where id=1").Scan(user.age, user.id, user.name, user.phone, user.sex)

    stmt, e := DB.Prepare("select * from user where id=?")
    query, e := stmt.Query(1)
    query.Scan()
}

func DeleteUser(user User) bool {
    //开启事务
    tx, err := DB.Begin()
    if err != nil {
        fmt.Println("tx fail")
    }
    //准备sql语句
    stmt, err := tx.Prepare("DELETE FROM user WHERE id = ?")
    if err != nil {
        fmt.Println("Prepare fail")
        return false
    }
    //设置参数以及执行sql语句
    res, err := stmt.Exec(user.id)
    if err != nil {
        fmt.Println("Exec fail")
        return false
    }
    //提交事务
    tx.Commit()
    //获得上一个insert的id
    fmt.Println(res.LastInsertId())
    return true
}

func InsertUser(user User) bool {
    //开启事务
    tx, err := DB.Begin()
    if err != nil {
        fmt.Println("tx fail")
        return false
    }
    //准备sql语句
    stmt, err := tx.Prepare("INSERT INTO user (`name`, `phone`) VALUES (?, ?)")
    if err != nil {
        fmt.Println("Prepare fail")
        return false
    }
    //将参数传递到sql语句中并且执行
    res, err := stmt.Exec(user.name, user.phone)
    if err != nil {
        fmt.Println("Exec fail")
        return false
    }
    //将事务提交
    tx.Commit()
    //获得上一个插入自增的id
    fmt.Println(res.LastInsertId())
    return true
}

func main() {
    InitDB()
    Query()
    defer DB.Close()
}
```