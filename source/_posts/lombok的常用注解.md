---
title: lombok的常用注解
toc_number: true
copyright: true
date: 2019-10-31 10:17:12
categories: lombok 
tags:
- lombok
---

# Lombok介绍

[官网]( https://projectlombok.org/ )介绍：

> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
> Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more. 

翻译：Lombok 项目是一个 Java 库，它会自动插入您的编辑器和构建工具中，从而使您的 Java 更加生动有趣。
永远不要再写另一个 getter 或 equals 方法，带有一个注释的您的类有一个功能全面的生成器，自动化您的日志记录变量等等。

简而言之就是：使用Lombok可以简化开发， 提高效率。

<!--more-->

# Lombok安装

导入maven依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.10</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
```

**注意**：由于 Lombok 是在编译时期才生成代码，因此需要在 IDE 上安装对应的插件避免 IDE 报错。

IDEA 上安装插件的流程是：

- Go to `File > Settings > Plugins`
- Click on `Browse repositories...`
- Search for `Lombok Plugin`
- Click on `Install plugin`
- Restart IntelliJ IDEA

eclipse 上安装请自行百度解决。

# Lombok常用注解

## @Getter/@Setter

 Never write `public int getFoo() {return foo;}` again. 

注解在类上或者字段上，为字段生成对应的 get/set 方法

## @ToString

 No need to start a debugger to see your fields: Just let lombok generate a `toString` for you! 

注解在类上，为类提供 toString() 方法，可以使用 exclude 排除不需要添加的字段和依赖

## @EqualAndHashCode

 Equality made easy: Generates `hashCode` and `equals` implementations from the fields of your object.. 

注解在类上，为类提供 equal() 和 hashCode() 方法，可以使用 exclude 排除不需要添加的字段和依赖

## @Data

 All together now: A shortcut for `@ToString`, `@EqualsAndHashCode`, `@Getter` on all fields, and `@Setter` on all non-final fields, and `@RequiredArgsConstructor`! 

注解在类上，与同时使用以下的注解的效果是一样的：

- **@ToString**
- **@Getter**
- **@Setter**
- **@RequiredArgsConstructor**
- **@EqualsAndHashCode**

## @NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor

 Constructors made to order: Generates constructors that take no arguments, one argument per final / non-nullfield, or one argument for every field. 

1. @NoArgsConstructor 注解为类生成一个无参的构造函数
2. @AllArgsConstructor 注解为类生成一个包含全部成员的构造函数
3. @RequiredArgsConstructor 注解为类提供一个必须初始化的成员的构造函数，如 final、static等

## @NonNull

 or: How I learned to stop worrying and love the NullPointerException. 

注解在字段或者方法的参数前，自动生成非空校验

## @Cleanup

 Automatic resource management: Call your `close()` methods safely with no hassle. 

 `@Cleanup` 注解用于自动管理资源，用在局部变量之前，在当前变量范围内即将执行完毕退出之前会自动清理资源，自动生成 `try-finally` 这样的代码来关闭流。 

如： **@Cleanup** InputStream in = **new** FileInputStream("...");        **@Cleanup** OutputStream out = **new** FileOutputStream("...."); 

## @Builder

 ... and Bob's your uncle: No-hassle fancy-pants APIs for object creation! 

注解在类上，为该类提供一个构造者模式

## @SneakyThrows

 To boldly throw checked exceptions where no one has thrown them before! 

注解在方法上，相当于使用 try/catch 捕获异常

## @Synchronized

 `synchronized` done right: Don't expose your locks. 

注解在方法上，相当于对该方法加了 Synchronized 同步锁

## @With

 Immutable 'setters' - methods that create a clone but with one changed field. 

注解在字段上，自动生成一个 `withFieldName(newValue)` 的方法，该方法会基于 newValue 调用相应构造函数，创建一个新的实例。 

## @Log

 Captain's Log, stardate 24435.7: "What was that line again?" 

注解在类上，为类提供一个属性名为 log 的日志对象

具体日志类型可以选择，类型如下

![image-20191031123715834](lombok%E7%9A%84%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3/image-20191031123715834.png)

## var

 Mutably! Hassle-free local variables. 

 var 用在局部变量前面， Lombok 在编译时还会自动进行类型推断。 

## val

 Finally! Hassle-free final local variables. 

 val 用在局部变量前面，相当于将变量声明为 final， Lombok 在编译时还会自动进行类型推断。 

# 注意事项

1. 在类需要序列化、反序列化时详细控制字段时（例如：Jackson json序列化）；
2. 使用 Lombok 能够省去手动创建 setter 和 getter 方法，但是也降低了源代码文件的可读性和完整性，降低了源代码阅读的舒适度；
3. 使用 @Slf4j 还是 @Log4j 看项目使用的日志框架；

4. 选择适合的地方使用 Lombok，例如 POJO 是一个好地方，因为它很单纯
5. 更多具体内容请以 [官方标准说明]( https://projectlombok.org/features/all ) 为主

# 参考资料

1.  https://juejin.im/post/5db9a6a65188251d5e7560bf#heading-44 









