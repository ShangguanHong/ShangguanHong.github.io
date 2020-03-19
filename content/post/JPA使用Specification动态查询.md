---
title: JPA使用Specification动态查询
date: 2019-07-22 16:28:17
copyright: true
categories: [SpringDataJpa学习笔记]
tags: ["Spring Data Jpa"]

---

# 1. 前言

有时我们在查询某个实体的时候，给定的条件是不固定，这个时候就需要动态构建相应的查询语句，在 Spring Data JPA 中可以通过继承 JpaSpecificationExecutor 接口来实现动态查询。相比于使用 JPQL 来进行动态查询，其优势是类型安全，更加的面向对象。

DAO 层需要继承 JpaSpecificationExecutor 接口

```java
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User>{}
```

项目代码：https://github.com/ShangguanHong/DemoSpringBoot/tree/master/springboot-jpa

# 2. 分析 JpaSpecificationExecutor 接口

JpaSpecificationExecutor 接口内的方法有以下几个，我们可以看到方法中都接受Specification(查询条件) 作为参数。

```java
public interface JpaSpecificationExecutor<T> { 
    // 根据条件查询单个对象
    Optional<T> findOne(@Nullable Specification<T> var1);
	// 根据条件查询列表
    List<T> findAll(@Nullable Specification<T> var1);
	// 根据条件查询列表并分页
    Page<T> findAll(@Nullable Specification<T> var1, Pageable var2);
	// 根据条件查询列表并排序
    List<T> findAll(@Nullable Specification<T> var1, Sort var2);
	// 满足条件的数量
    long count(@Nullable Specification<T> var1);
}
```

>`Specification<T>` ：查询条件
>
>`Pageable` ：分页参数
>
>`Sort` ：排序参数

# 3. 分析 Specification 接口

打开 Specification 接口可以看到里面就定义了一个方法

```java
// root：查询的根对象，查询的任何属性都可以从根对象中获取（类似 SQL 语句 from 后面的表）
// query：顶层查询对象，自定义查询方式（一般不用到）
// cb：查询的构造器，封装了很多的查询条件(精确查询，模糊查询等)
// 比较的属性在 root 中，比较的方式在 cb 中
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
```

由于 Specification 是一个接口，因此 **我们想自定义查询条件，就需要自定义一个我们自己的 Specification 实现类**

# 4. 使用示例

1. 精确查询

```java
    // 查询用户名为root的用户
    @Test
    @Transactional
    public void SpecTest1() {
        // lambda 表达式
        User user = userRepository.findOne((root, query, cb) -> {
            // 比较方式为精确查询：cb.equal
            // 比较的属性为用户名：root.get("userName")
            // 比较的类型：as(String.class)
            Predicate predicate = cb.equal(root.get("userName").as(String.class), "root");
            return predicate;
        }).get();
        System.out.println(user);
    }
```

![1563790233042](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1563790233042.png)

2. 模糊查询

```java
	// 查询用户名为test开头的用户
    @Test
    @Transactional
    public void SpecTest2() {
        // lambda 表达式
        List<User> users = userRepository.findAll((root, query, cb) -> {
            // 比较方式为模糊查询：cb.like
            // 比较的属性为用户名：root.get("userName")
            // 比较的类型：as(String.class)
            Predicate predicate = cb.like(root.get("userName").as(String.class), "test%");
            return predicate;
        });
        for (User user : users) {
            System.out.println(user);
        }
    }
```

![1563790782009](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1563790782009.png)

3. 多条件查询

```java
    // 查询用户名为test开头并且地址为shanghai的用户
    @Test
    @Transactional
    public void SpecTest3() {
        // lambda 表达式
        List<User> users = userRepository.findAll((root, query, cb) -> {
            // 比较方式为模糊查询：cb.like
            // 比较的属性为用户名：root.get("userName")
            // 比较的类型：as(String.class)
            List<Predicate> list = new ArrayList<>();
            list.add(cb.like(root.get("userName").as(String.class), "test%"));
            // 比较方式为精确查询：cb.equal
            // 比较的属性为地址：root.get("userAddr")
            // 比较的类型：as(String.class)
            list.add(cb.equal(root.get("userAddr").as(String.class), "shanghai"));
            Predicate[] p = new Predicate[list.size()];
            // cb.and ： 所有条件的与(或用or)
            return cb.and(list.toArray(p));
        });
        for (User user : users) {
            System.out.println(user);
        }
    }
```

![1563792083275](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1563792083275.png)

4. 排序查询

```java
    // 查询所有用户，并以用户ID倒序
    @Test
    @Transactional
    public void SpecTest4() {
        // 创建排序对象，需要调用构造方法实例化sort对象
        // 第一个参数：排序的顺序（Sort.Direction.ASC:正序，Sort.Direction.DESC：逆序）
        // 第二个参数：排序的属性名称
        Sort sort = new Sort(Sort.Direction.DESC, "id");
        // lambda 表达式
        // 排序查询
        List<User> users = userRepository.findAll((root, query, cb) -> {
            // 不做条件筛选
            return null;
        }, sort);
        for (User user : users) {
            System.out.println(user);
        }
    }
```

![1563792658039](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1563792658039.png)

5. 分页查询

```java
    // 查询所有用户，并进行分页
    @Test
    @Transactional
    public void SpecTest5() {
        // 创建分页对象，PageRequest对象是Pageable接口的实现类
        // 调用PageRequest的构造函数实例化一个PageRequest对象
        // 第一个参数：当前查询的页数（从0开始）
        // 第二个参数：每页的数量
        Pageable pageable = new PageRequest(0, 1);
        // lambda 表达式
        // 分页查询
        Page<User> userPage = userRepository.findAll((root, query, cb) -> {
            // 不做条件筛选
            return null;
        }, pageable);
        List<User> users = userPage.getContent();
        System.out.println(users);
        System.out.println("用户总数量为：" + userPage.getTotalElements());
        System.out.println("总页数为:" + userPage.getTotalPages());
    }
```

![1563793110699](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1563793110699.png)

