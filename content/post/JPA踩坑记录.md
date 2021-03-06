---
title: JPA踩坑记录
date: 2019-07-19 10:22:40
copyright: true
categories: [常见错误分类]
tags: [Jpa]

---

**以下是本人使用 JPA 的时候遇到的坑，记录一下**

1. 在使用的时候,能使用单向关联就尽量不要用双向关联,如果使用双向关联，请记得一定要在非主控端加上 @JsonIgnore 等让 Json 能够忽略序列化的注解，否则可能会出现爆内存的提示
2. JPA 中的双向 @ManyToMany 不建议使用，尽量能不双向就不双向使用 @ManyToMany，如果非要使用千万记得要在非主控端加上 @JsonIgnore 等让 Json 能够忽略序列化的注解，否则可能会出现爆内存的错误提示
3. 使用 findById() 而不是 getOne() 来通过主键查找一个实例。因为findById()是立即加载而getOne() 是使用延迟加载，需要在方法上加上@Transactional 注解，否则会出现懒加载异常
4. JPA 自定义增删改的时候,记得加 @Transactional 注解，否则会出现懒加载异常
5. 如果要使用懒加载的话,需要在 service 层增加 @Transactional 注解,以避免 session 被误关闭造成懒加载异常
6. 不要相信 @DynamicUpdate 注解会自己给你 pass 掉属性为 null 的字段,它依旧会把数据库里所有你没传入参数的值设置为 null
7. 想要动态更新的最好方式是将数据库中该条记录所有信息全部查询出来,并使用 copyProperties 将不为 null 的字段复制到新修改的字段中
8. 在 @Test 下测试 Spring Boot JPA 时，要加上 @Transactional 与 @RollBack(value= false) 注解，否则执行完测试后，JPA会回滚操作

如果还遇到什么问题，会持续更新...

