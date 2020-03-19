---
title: Java中的对象术语(PO/POJO/VO/BO/DAO/DTO)
date: 2019-07-17 20:09:34
copyright: false
categories: [Java学习笔记]
tags: [Java]

---



本文转载自 [Java中的对象术语(PO/POJO/VO/BO/DAO/DTO)](https://blog.csdn.net/u010297957/article/details/49817563)



## **1. PO(persistant object) : 持久对象**

1. 理解为dao层中接收和返回的java bean，也就是通常写在model包中的model

2. 有时也被称为Data对象，对应数据库中的entity，可以简单认为一个PO对应数据库中的一条记录，多个记录可以用PO的集合。

3. 在o/r 映射的时候出现的概念,如果没有o/r映射,就没有这个概念存在了。

4. PO中应该不包含任何对数据库的操作。

## **2. VO(value object): 值对象 / view object表现层对象**

1. 理解为view层中用于显示的java bean

2. 主要对应页面显示（web页面(jsp...)/swt、swing界面）的数据对象，所以它可以和表对应，也可以不（大部分情况是表所有字段集合的子集），这根据业务的需要。

3. 与DTO的区别是：DTO用于无界面的web service传输中而VO用于界面的展示，可以把DTO转化为VO提供给前台。

4. 例如在struts中，用ActionForm做VO，需要做一个转换，因为PO是面向对象的，而ActionForm是和view对应的，要将几个PO要显示的属性合成一个ActionForm，可以使用BeanUtils的copy方法。

## **3. BO(business object): 业务对象**

1. 理解为service层中接收和返回的java bean

2. 从业务模型的角度看,见UML元件领域模型中的领域对象.封装业务逻辑的java对象,通过调用DAO方法,结合PO,VO进行业务操作。

3. 根据业务逻辑，将封装业务逻辑为一个对象，可以包括多个PO，通常需要将BO转化成PO，才能进行数据的持久化，反之，从DB中得到的PO，需要转化成BO才能在业务层使用。

4.  关于BO主要有三种概念
   1.  只包含业务对象的属性；  
   2.  只包含业务方法； 
   3.  两者都包含。

## **4. POJO(plain ordinary java object) :简单无规则java对象**

1. 理解为各个层中接收和返回的java bean统称

2. 抽象的统一概念，一个中间对象，可以转化为PO、DTO、VO（或者说PO、DTO是POJO的不同的具体阶段的名字）。

3. POJO持久化之后==〉PO（在运行期，由Hibernate中的cglib动态把POJO转换为PO，PO相对于POJO会增加一些用来管理数据库entity状态的属性和方法。PO对于programmer来说完全透明，由于是运行期生成PO，所以可以支持增量编译，增量调试。）

4. POJO传输过程中==〉DTO

5. POJO用作表示层==〉VO

## **5. DAO(data access object): 数据访问对象**

1. 是sun的一个标准j2ee设计模式,这个模式中有个接口就是DAO，负责将PO持久化到数据库，也负责将数据库查询的结果集映射为PO。

2. 为业务层提供接口.此对象用于访问数据库（CRUD操作）.通常和PO结合使用,DAO中包含了各种数据库的操作方法.通过它的方法,结合PO对数据库进行相关的操作.夹在业务逻辑与数据库资源中间.配合VO, 提供数据库的CRUD操作。

## **6. DTO (Data Transfer Object):数据传输对象**

1. 理解为controller层中接收和返回的java bean

2. 用在需要跨进程或远程传输时，它不应该包含业务逻辑。

3. 比如一张表有100个字段，那么对应的PO就有100个属性（**大多数情况下，DTO 内的数据来自多个表**）。但view层只需显示10个字段，没有必要把整个PO对象传递到client，这时我们就可以用只有这10个属性的DTO来传输数据到client，这样也不会暴露server端表结构。到达客户端以后，如果用这个对象来对应界面显示，那此时它的身份就转为VO。

