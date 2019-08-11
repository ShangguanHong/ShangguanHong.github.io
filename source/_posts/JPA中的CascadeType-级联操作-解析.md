---
title: JPA中的CascadeType(级联操作)解析
date: 2019-07-19 12:27:11
copyright: true
categories: Jpa学习笔记
tags:
- Jpa
---

# 1. Cascade介绍

​		Cascade(级联)在编写触发器时经常用到，触发器的作用是当 主控表信息改变时，用来保证其关联表中数据同步更新。若对触发器来修改或删除关联表相记录，必须要删除对应的关联表信息，否则，会存有脏数据。所以，适当的做法是，删除主表的同时，关联表的信息也要同时删除，在 hibernate 中，只需设置 cascade 属性值即可。

下面就介绍一下 cascade 的各种属性值。

<!--more--> 

# 2. CascadeType.PERSIST

级联新增（保存）操作：持久保存一个实体时，也会持久保存该实体的所有相关实体。

下面举个例子来解释一下。

```java
@Entity
@Data
@Table(name = "sys_user")
public class User {
    @ManyToMany(cascade = {CascadeType.PERSIST}, fetch = FetchType.EAGER)
    @JoinTable(name = "sys_user_role",
            joinColumns = {@JoinColumn(name = "u_id", referencedColumnName = "id")},
            inverseJoinColumns = {@JoinColumn(name = "r_id", referencedColumnName = "id")})
    private Set<Role> roles = new HashSet<>();
}

```

>​		可以看到上面的代码中，我们给予了 user 表对于 role 表的级联保存权限，这可以做到 **当我要保存一个用户的时候，系统会将此用户拥有的角色也一起保存到数据库中** ，这便是级联保存
>
>**注意：** 如果用户的角色已经存在数据库中，那么在执行级联保存的时候将会报错，这时就需要与级联合并(Cascade.MERGE)一起使用

# 3. Cascade.MERGE

级联更新（合并）操作：持久保存一个实体时，会持久更新该实体的所有相关实体。

>当 user 的 roles 里的字段发生变化时，会将此变化更新到 role 表中
>
>**注意：** 如果对用户的角色进行了增加，那么在执行级联合并的时候(因为并没有此角色)将会报错，这时就需要与级联保存(Cascade.PERSIST)一起使用

# 4. Cascade.REMOVE

级联删除操作：持久删除一个实体时，与它有映射关系的实体也会跟着被删除。

>当我们删除一个用户时，会将该用户拥有的所有角色全部删除
>
>**注意：** 如果你删除的 用户里的角色不止你一个用户拥有时(一对一与一对多就没此问题)，删除此用户时级联删除会报错(因为别的用户还需要那些角色)，因此多对多时慎用级联删除操作

# 5. Cascade.DETACH

级联游离(托管)操作：当删除一个实体时，若该实体有外键关联无法删除时，该权限将撤销所有的外键关联，将此实体删除

# 6. Cascade.REFRESH

级联刷新操作：获取一个实体对象的同时也会重新获取最新的关联对象。

# 7. Cascade.ALL

拥有上述五种权限。

# 8. 参考资料

1.  [Hibernate 注解中CascadeType用法汇总](https://www.cnblogs.com/printN/p/6409330.html)





