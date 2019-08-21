---
title: 如何在Java8中处理日期和时间
copyright: true
date: 2019-08-15 22:11:56
categories: Java学习笔记
tags:
- Java8
---

# 1. 前言

在 Java 8 之前处理日期时间的 `API` 存在着一些诸如非线程安全、设计差、时区处理麻烦等问题，为了解决这一系列的问题，Java 8 推出了全新的日期时间 `API` 供使用，包路径为 `java.time`

其中最常使用的为下面的几个类

- `Instant`：瞬时实例
- `LocalDate`：本地日期，不包含具体时间 
- `LocalTime`：本地时间，不包含日期
- `LocalDateTime`：组合了日期和时间，但不包含时差和时区信息。
- `ZonedDateTime`：最完整的日期时间，包含时区和相对`UTC`或格林威治的时差。

下面将举出具体例子来详细介绍

<!--more-->

# 2. 实例

## 1. 获取今天的日期

Java 8 中的 `LocalDate` 用于表示当天日期。和 `java.util.Date` 不同，它只有日期，不包含时间。当你仅需要表示日期时就用这个类。

```java
LocalDate now = LocalDate.now(); // 2019-08-15
```

上面的代码创建了当天的日期，不含时间信息。打印出的日期格式非常友好，不像老的 Date 类打印出一堆没有格式化的信息。

## 2. 获取年、月、日信息

`LocalDate` 类提供了获取年、月、日的快捷方法，其实例还包含很多其它的日期属性。通过调用这些方法就可以很方便的得到需要的日期信息，不用像以前一样需要依赖 `java.util.Calendar` 类了

```java
LocalDate now = LocalDate.now(); // 2019-08-15
// 获取年份 2019
int year = now.getYear();
// 获取月份 8
int monthValue = now.getMonthValue();
// 从月份中获取天数 15
int dayOfMonth = now.getDayOfMonth();
```

## 3. 处理特定日期

在第一个例子里，我们通过静态工厂方法 now() 非常容易地创建了当天日期，另外还可以调用另一个静态工厂方法 `LocalDate.of()` 创建任意日期， 该方法需要传入年、月、日做参数，返回对应的 `LocalDate` 实例。这个方法的好处是没再犯老 `API` 的设计错误，比如年度起始于 1900，月份是从 0 开始等等。日期所见即所得，就像下面这个例子表示了 8 月 15 日，没有任何隐藏机关。

```java
LocalDate date = LocalDate.of(2019, 8, 15);
System.out.println(date); // 2019-08-15
```

可以看到创建的日期完全符合预期，与写入的 2018 年 6 月 20 日完全一致。

## 4. 判断两个日期是否相等

现实生活中有一类时间处理就是判断两个日期是否相等。你常常会检查今天是不是个特殊的日子，比如生日、纪念日或非交易日。这时就需要把指定的日期与某个特定日期做比较，例如判断这一天是否是假期。

```java
LocalDate now = LocalDate.now();
LocalDate date = LocalDate.of(2019, 8, 15);
if (date.equals(now)) {
	System.out.println("同一天");
}
```

## 5. 检查周期性事件

Java 中另一个日期时间的处理就是检查类似每月账单、结婚纪念日、`EMI`日或保险缴费日这些周期性事件。如果你在电子商务网站工作，那么一定会有一个模块用来在圣诞节、感恩节这种节日时向客户发送问候邮件。Java 中如何检查这些节日或其它周期性事件呢？答案就是 `MonthDay` 类。这个类组合了月份和日，去掉了年，这意味着你可以用它判断每年都会发生事件。和这个类相似的还有一个 `YearMonth` 类。这些类也都是不可变并且线程安全的值类型。下面我们通过 `MonthDay` 来检查周期性事件：

```java
LocalDate now = LocalDate.now();
LocalDate dateOfBirth = LocalDate.of(2019, 8, 15);
// 根据月份和日期创建 YearMonth
MonthDay birthday = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
// 直接从一个 LocalDate 实例创建 YearMonth
MonthDay currentMonthDay = MonthDay.from(now);
if (currentMonthDay.equals(birthday)) {
	System.out.println("Happy Birthday");
} else {
	System.out.println("Sorry, today is not your birthday");
}
```

## 6. 获取当前时间

与 Java 8 获取日期的例子很像，获取时间使用的是 `LocalTime` 类，一个只有时间没有日期的 `LocalDate` 近亲。可以调用静态工厂方法 now() 来获取当前时间。默认的格式是 `hh:mm:ss:nnn`。

```java
LocalTime localTime = LocalTime.now(); 
System.out.println(localTime); // 22:58:04.238
```

可以看到当前时间就只包含时间信息，没有日期。

## 7. 在现有的时间上增加小时

通过增加小时、分、秒来计算将来的时间很常见。Java 8 除了不变类型和线程安全的好处之外，还提供了更好的 `plusHours()` 方法替换 add()，并且是兼容的。注意，这些方法返回一个全新的 `LocalTime` 实例，由于其不可变性，返回后一定要用变量赋值。

```java
LocalTime localTime = LocalTime.now();
System.out.println(localTime); // 23:00:25.913
LocalTime localTime1 = localTime.plusHours(2);//增加2小时
System.out.println(localTime1); // 01:00:25.913
```

可以看到，新的时间在当前时间 `23:00:25.913` 的基础上增加了 2 个小时。

## 8. 计算一周后的日期

和上个例子计算两小时以后的时间类似，这个例子会计算一周后的日期。`LocalDate` 日期不包含时间信息，它的 plus() 方法用来增加天、周、月，`ChronoUnit` 类声明了这些时间单位。由于 `LocalDate` 也是不变类型，返回后一定要用变量赋值。

```java
LocalDate now = LocalDate.now();
System.out.println(now); // 2019-08-15
LocalDate plusDate = now.plus(1, ChronoUnit.WEEKS);
System.out.println(plusDate); // 2019-08-22

```

可以看到新日期离当天日期是 7 天，也就是一周。你可以用同样的方法增加 1 个月、1 年、1 小时、1 分钟甚至一个世纪，更多选项可以查看 Java 8 `API` 中的 `ChronoUnit` 类。

## 9. 计算一年前或一年后的日期

继续上面的例子，上个例子中我们通过 `LocalDate` 的 plus() 方法增加天数、周数或月数，这个例子我们利用 minus() 方法计算一年前的日期。

```java
LocalDate now = LocalDate.now();
LocalDate minusDate = now.minus(1, ChronoUnit.YEARS);
LocalDate plusDate1 = now.plus(1, ChronoUnit.YEARS);
System.out.println(now); // 2019-08-15
System.out.println(minusDate); // 2018-08-15
System.out.println(plusDate1); // 2020-08-15
```

## 10. Clock 时钟类

Java 8 增加了一个 Clock 时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到 `System.currentTimeInMillis()` 和 `TimeZone.getDefault()` 的地方都可用 Clock 替换。

```java
System.out.println(Clock.systemUTC().millis()); // 1565881666099
System.out.println(System.currentTimeMillis()); // 1565881666099
System.out.println(Clock.systemUTC()); // SystemClock[Z]
System.out.println(Clock.systemDefaultZone()); // SystemClock[Asia/Shanghai]
```

## 11. 判断日期是早于还是晚于另一个日期

另一个工作中常见的操作就是如何判断给定的一个日期是大于某天还是小于某天？在 Java 8 中，`LocalDate` 类有两类方法 `isBefore()` 和 `isAfter()` 用于比较日期。调用 `isBefore()` 方法时，如果给定日期小于当前日期则返回 true。

```java
LocalDate tomorrow = LocalDate.of(2019, 8, 15);
if (tomorrow.isAfter(now)) {
	System.out.println("Tomorrow comes after today");
}
LocalDate yesterday = now.minus(1, ChronoUnit.DAYS);
if (yesterday.isBefore(now)) {
	System.out.println("Yesterday is day before today");
}
```

在 Java 8 中比较日期非常方便，不需要使用额外的 Calendar 类来做这些基础工作了。

## 12. 处理时区

Java 8 不仅分离了日期和时间，也把时区分离出来了。现在有一系列单独的类如 `ZoneId` 来处理特定时区，`ZoneDateTime` 类来表示某时区下的时间。这在 Java 8 以前都是 `GregorianCalendar` 类来做的。

```java
ZoneId america = ZoneId.of("America/New_York");
LocalDateTime localDateAndTime = LocalDateTime.now();
ZonedDateTime dateAndTimeInNewYork  = ZonedDateTime.of(localDateAndTime, america);
System.out.println(dateAndTimeInNewYork); 
// 2019-08-15T23:16:35.572-04:00[America/New_York]
```

## 13. 表示固定年月

与 `MonthDay` 检查重复事件的例子相似，`YearMonth` 是另一个组合类，用于表示信用卡到期日、FD 到期日、期货期权到期日等。还可以用这个类得到 当月共有多少天，`YearMonth` 实例的 `lengthOfMonth()` 方法可以返回当月的天数，在判断 2 月有 28 天还是 29 天时非常有用。

```java
System.out.printf("Days in month year %s: %d%n", yearMonth, yearMonth.lengthOfMonth()); 
// Days in month year 2019-08: 31
YearMonth creditCardExpiry = YearMonth.of(2020, Month.FEBRUARY);
System.out.printf("Your credit card expires on %s %n", creditCardExpiry); 
// Your credit card expires on 2020-02 
```

## 14. 检查闰年

`LocalDate` 类有一个很实用的方法 `isLeapYear()` 判断该实例是否是一个闰年。

```java
LocalDate localDate = LocalDate.now();
if (localDate.isLeapYear()) {
	System.out.println("is leap year");
}
```

## 15. 计算两个日期之间的天数和月数

有一个常见日期操作是计算两个日期之间的天数、周数或月数。在 Java 8 中可以用 `java.time.Period` 类来做计算。下面这个例子中，我们计算了当天和将来某一天之间的月数。

```java
LocalDate date = LocalDate.of(2020, Month.MARCH, 20);
Period period = Period.between(now, date);
System.out.println("离下个时间还有" + period.getMonths() + " 个月");
```

## 16. 包含时差信息的日期和时间

在 Java 8 中，`ZoneOffset` 类用来表示时区，举例来说印度与 `GMT` 或 `UTC` 标准时区相差 `+05:30`，可以通过 `ZoneOffset.of()` 静态方法来获取对应的时区。一旦得到了时差就可以通过传入 `LocalDateTime` 和 `ZoneOffset` 来创建一个 `OffSetDateTime` 对象。

```java
LocalDateTime datetime = LocalDateTime.of(2019, 8, 15, 23, 30);
ZoneOffset offset = ZoneOffset.of("+05:30");
OffsetDateTime date = OffsetDateTime.of(datetime, offset);  
System.out.println("Date and Time with timezone offset in Java : " + date);
// Date and Time with timezone offset in Java : 2019-08-15T23:30+05:30
```

## 17. 获取当前的时间戳

如果你还记得 Java 8 以前是如何获得当前时间戳，那么现在你终于解脱了。`Instant` 类有一个静态工厂方法 `now()` 会返回当前的时间戳，如下所示：

```java
Instant timestamp = Instant.now();
System.out.println(timestamp); // 2019-08-15T15:30:01.555Z
```

时间戳信息里同时包含了日期和时间，这和 `java.util.Date` 很像。实际上 Instant 类确实等同于 Java 8 之前的 Date类，你可以使用 Date 类和 Instant 类各自的转换方法互相转换，例如：`Date.from(Instant)` 将 Instant 转换成 `java.util.Date`，`Date.toInstant()` 则是将 Date 类转换成 Instant 类。

## 18. 使用预定义的格式化工具去解析或格式化日期

在 Java 8 以前的世界里，日期和时间的格式化非常诡异，唯一的帮助类 `SimpleDateFormat` 也是非线程安全的，而且用作局部变量解析和格式化日期时显得很笨重。幸好线程局部变量能使它在多线程环境中变得可用，不过这都是过去时了。Java 8 引入了全新的日期时间格式工具，线程安全而且使用方便。它自带了一些常用的内置格式化工具。

```java
LocalDate date = LocalDate.parse("2019-08-15");
System.out.println(date); // 2019-08-15
LocalTime time = LocalTime.parse("23:45:20");
System.out.println(time); // 23:45:20
```

## 19. 使用自定义格式化工具解析日期

尽管内置格式化工具很好用，有时还是需要定义特定的日期格式。可以调用 `DateTimeFormatter` 的 `ofPattern()` 静态方法并传入任意格式返回其实例，格式中的字符和以前代表的一样，M 代表月，m 代表分。如果格式不规范会抛出 `DateTimeParseException` 异常，不过如果只是把 M 写成 m 这种逻辑错误是不会抛异常的。

```java
DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyyy MM dd");
String s = "2019 08 15";
LocalDate localDate = LocalDate.parse(s, pattern);
System.out.println(localDate); // 2019-08-15
```

## 20. 把日期转换成字符串

上两个主要是从字符串解析日期。现在我们反过来，把 `LocalDateTime` 日期实例转换成特定格式的字符串。这是迄今为止 Java 日期转字符串最为简单的方式了。下面的例子将返回一个代表日期的格式化字符串。和前面类似，还是需要创建 `DateTimeFormatter` 实例并传入格式，但这回调用的是 format() 方法，而非 parse() 方法。这个方法会把传入的日期转化成指定格式的字符串。

```java
LocalDateTime arrivalDate = LocalDateTime.now();
	try {
		DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy MM dd  hh:mm:ss");
		String landing = arrivalDate.format(format);
		System.out.printf("Arriving at :  %s %n", landing);
        // Arriving at :  2019 08 15  11:36:37 
	} catch (DateTimeException ex) {
		System.out.printf("%s can't be formatted!%n", arrivalDate);
		ex.printStackTrace();
	}
```

# 参考资料

1. [20 个案例教你在 Java 8 中如何处理日期和时间?](http://www.54tianzhisheng.cn/2018/06/20/java-8-date/)