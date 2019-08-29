---
title: Spring Boot中CommandLineRunner的使用
copyright: true
date: 2019-08-29 21:24:02
categories: SpringBoot学习记录
tags:
- Spring Boot
---

# 前言

在项目中经常会遇到需要在项目启动之前做一些初始化数据的操作，例如初始化线程池、初始化一些脚本等等。但是往往这些操作又需要在 `Spring Beans` 加载完成之后，因为需要依赖于一些 `Spring Beans`。`CommandLineRunner` 可以很好的解决这个问题，继承了 `ComandLineRunner` 接口并且加上了 `@Component` 注解的类可以在  `Spring Beans`  都初始化之后，而又在 `SpringApplication.run()`  之前执行。下面来详细介绍 `CommandLineRunner` 的使用。

<!--more-->

# 正文

修改启动类，添加两行打印代码，看查看 `CommandLineRunner`  运行的时机

```java
@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        System.out.println("This Application run before");
        SpringApplication.run(TestApplication.class, args);
        System.out.println("This Application run after");
    }
}
```

然后写一个 `Runner` 类，继承 `ComandLineRunner` 接口，并实现 `run()` 方法

```java
@Component
public class Runner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        for (int i = 0; i < 5; i++) {
            System.out.println("Runner no." + i);
        }
    }
}
```

**注意** ：一定要加上 `@Component` 注解

我们启动应用查看输出，输出顺序如下（省略不重要的）

```shell
This Application run before

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.7.RELEASE)
 
2019-08-29 21:51:55.532  INFO 5576 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-08-29 21:51:55.555  INFO 5576 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-08-29 21:51:55.555  INFO 5576 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2019-08-29 21:51:55.660  INFO 5576 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-08-29 21:51:55.660  INFO 5576 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1361
Runner no.0
Runner no.1
Runner no.2
Runner no.3
Runner no.4
This Application run after
```

可以看到 `run()` 方法确实是在 `Spring Beans` 都加载完成后，但是又在 `SpringApplication.run()`  之前执行。

那么如果有多个类继承了 `CommandLineRunner`  接口并且加上了 `@Component` 注解，那么执行顺序如何呢？

`Spring Boot` 提供一个注解 `@Order` 来解决这个问题，@Order 注解内有一个 `value` 值，该值设置的越小，则该类越早被执行。

`RunnerOrder1.class`

```java
@Component
@Order(value = 1)
public class RunnerOrder1 implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println(this.getClass().getName());
    }
}
```

`RunnerOrder2.class`

```java
@Component
@Order(value = 2)
public class RunnerOrder2 implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println(this.getClass().getName());
    }
}
```

执行结果如下

```shell
This Application run before

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.7.RELEASE)

2019-08-29 22:06:31.538  INFO 13368 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-08-29 22:06:31.566  INFO 13368 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-08-29 22:06:31.566  INFO 13368 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.22]
2019-08-29 22:06:31.680  INFO 13368 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-08-29 22:06:31.681  INFO 13368 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1617 ms
com.example.mybatisplus.service.component.RunnerOrder1
com.example.mybatisplus.service.component.RunnerOrder2
This Application run after
```

那么如果两个类的执行优先级一致呢？本人的测试结果是：**对于执行优先度同一级的类，按类的全限定类名的字符串排序，字典序越小的越早被执行。**

对于上述的需求，还有一种方法是实现 `ApplicationRunner` 接口，用法与上述一致，唯一的区别就是 `ApplicationRunner`  的 run() 方法传入的参数是 `ApplicationArguments`，而 `CommandLineRunner`  的 run() 方法传入的是 String 数组，如果你想要得到更加详细传入信息就可以使用 `ApplicationRunner` 。