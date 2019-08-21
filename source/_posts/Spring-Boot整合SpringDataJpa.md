---
title: Spring Boot整合Spring Data Jpa
date: 2019-07-08 14:14:55
copyright: true
categories: 
- [SpringBoot学习笔记]
- [SpringDataJpa学习笔记]
tags:
- Spring Boot
- Spring Data Jpa
---

# 1. 前言

之前的文章中写到了如何使用 Spring Boot 整合 mybatis(传送门: [Spring-Boot整合MyBatis](https://shangguanhong.github.io/2019/06/03/Spring-Boot整合MyBatis/))，今天学习一下如何使用 Spring Boot 整合 Spring Data Jpa。

项目代码:  https://github.com/ShangguanHong/DemoSpringBoot/tree/master/springboot-jpa

# 2. Spring Data JPA简介

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范封装的一套 JPA 应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能， 且易于扩展！学习并使用 Spring Data JPA 可以极大的提供开发效率。

Spring Data JPA 让我们解脱了 DAO 层 的操作，基本上所有 CRUD 都可以依赖于它来实现，在实际的工作工程中，推荐使用 Spring Data JPA + ORM (如: hibernate) 完成操作，这样在切换不同的 ORM 框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。

<!--more-->

# 3. Spring Boot整合Spring Data Jpa

## 3.1 导入依赖

1. SpringDataJpa依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```

2. MySQL连接驱动

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

## 3.2 配置文件

```yaml
server:
    port: 8080
    servlet:
        context-path: /
spring:
    datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/demo?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&useSSL=false
        username: root
        password: root
    jpa:
        database: MySQL
        database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
        show-sql: true
        hibernate:
            ddl-auto: update
```

> show-sql：控制台显示SQL语句
>
> ddl-auto可以有以下几种取值
>
> -  `create`：每次运行程序时，都会重新创建表，这会导致数据丢失
> -  `create-drop`：每次运行程序时会先创建表结构，在程序结束时清空表
> -  `upadte`：每次运行程序，没有表时会创建表，如果对象发生改变会更新表结构，原有数据不会清空，只会更新（推荐使用这种）
> -  `validate`：运行程序会校验数据与数据库的字段类型是否相同，字段不同会报错
> -  `none`: 禁用DDL处理，即什么都不做

## 3.3 编写Entity层

User.java

```java
package com.example.domain;

import lombok.Data;
import javax.persistence.*;

@Entity
@Data
@Table(name = "sys_user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "user_name")
    private String userName;

    @Column(name = "user_addr")
    private String userAddr;
    
    @Column(name = "email")
    private String Email;
}

```

>- `@Data`：Lombok 的注解，可以使我们省略 POJO的 Getter/Setter 的编写
>
>- `@Entity`：表示该类是一个实体类
>
>- `@Table`：配置表与实体类的映射关系，name 代表数据库表的名称
>
>- `@Id`：注解在主键上，一个实体只能有一个 @Id 注解
>
>- `@GeneratedValue`：strategy 属性表示主键的生成策略
>
>  1. GenerationType.IDENTITY：自增(底层数据库必须支持自动增长，例如 MySQL)， 最常用的一种
>
>  2. GenerationType.SEQUENCE：序列(底层数据库必须支持序列，例如 Oracle)
>
>  3. GenerationType.TABLE：jpa 提供的一种机制，通过一张表的形式帮助我们完成主键的自增
>
>  4. GenerationType.AUTO：由程序自动帮我们选择一种主键的生成策略
>
>- `@Column`：配置属性与字段的映射关系，name 代表字段的名称

## 3.5 编写DAO层

UserRepository.java

``` java
package com.example.repository;

import com.example.domain.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {

}

```

> - 只需要编写 DAO 层的接口，并不需要编写 DAO 层接口的实现类
> - DAO 层接口必须继承 JpaRepository接口（SpringDataJPA 提供的简单数据操作接口，基本的CRUD操作）和JpaSpecificationExecutor（SpringDataJPA 提供的复杂查询接口，例如分页等）。
> - JpaRepository 接口需要两个泛型，分别是操作的实体类的类型和实体类中主键属性的类型
> - JpaSpecificationExecutor 接口需要提供一个泛型，为操作的实体类的类型

## 3.6 基础方法

Spring Boot Jpa 默认预先生成了一些基本的 CRUD 的方法，例如：增、删、改等等

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserDaoTest {

    @Autowired
    private UserRepository userRepository;

    public void testUserDaoBaseOperate() {
        User user = new User();
        // 查找所有用户
        userRepository.findAll();
        // 根据id查找用户
        userRepository.findById(1l);
        // 保存/更新用户信息
        userRepository.save(user);
        // 删除用户
        userRepository.delete(user);
        // 统计用户数量
        userRepository.count();
        // 通过id查找用户是否存在
        userRepository.existsById(1l);
        // ...
    }
}

```

>Spring Data Jpa 的更新和新建都由 save 方法来实现，根据传入 save的实体类是否带有主键来进行判断，含有主键为更新操作，无主键进行新建操作

## 3.7 自定义简单查询

自定义的简单查询就是根据方法名来自动生成 SQL，需要定义在 DAO 层，这样才能自动生成实现方法。主要的语法是 `findXXBy` , `readAXXBy` , `queryXXBy` , `countXXBy` ,  `getXXBy` 后面跟属性名称：

```java
User findByUserName(String userName);
```

也可以使用一些关键字 `And ` 、 `Or`

```java
User findByUserNameOrEmail(String userName, String email);
```

修改、删除、统计也是类似语法

```java
Long deleteById(Long id);
Long countByUserName(String userName)
```

基本上 SQL 体系中的关键词都可以使用，例如：` LIKE `、 `IgnoreCase`、 `OrderBy`。

```java
List<User> findByEmailLike(String email);
User findByUserNameIgnoreCase(String userName);
List<User> findByUserNameOrderByEmailDesc(String email);
```

**具体的关键字，使用方法和生产成SQL如下表所示**

| Keyword           | Sample                                  | JPQL snippet                                                 |
| :---------------- | :-------------------------------------- | :----------------------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                                           |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1                              |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %)   |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                                      |
| TRUE              | findByActiveTrue()                      | … where x.active = true                                      |
| FALSE             | findByActiveFalse()                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)                        |



# 4. 使用 JPQL 自定义语句

有时我们想使用自定义的 JPQL 来查询，Spring Data JPA 也是完美支持的，编写在 DAO 中。不懂 JPQL 的可以先了解一下(传送门：[JPQL详解](https://blog.csdn.net/dtttyc/article/details/80001826))

```java
@Query(value = "from User where userName = ?1")
User findByUserName(String userName);

@Modifying
@Query(value = "update User set userName = ?1 where id = ?2")
int modifyByIdAndUserId(String userName, Long id);
	
@Transactional
@Modifying
@Query(value = "delete from User where id = ?1")
void deleteByUserId(Long id);
```

>- 在 查询方法上面使用 @Query 注解，value 为需要执行的 JPQL 语句
>- 如果涉及到删除和修改还需要加上 @Modifying 
>- 也可以根据需要添加 @Transactional 对事务的支持

# 5. 参考资料

1. [Spring Boot(五)：Spring Boot Jpa 的使用](http://www.ityouknow.com/springboot/2016/08/20/spring-boot-jpa.html)