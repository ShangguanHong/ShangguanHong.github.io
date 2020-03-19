---
title: Java常用工具类之StringUtils类
date: 2019-08-14 22:45:28
categories: [Java学习笔记]
tags: [java]
---

原文链接：https://blog.csdn.net/diypp2012/article/details/82971716



该工具类是用于操作 Java.lang.String 类的。
StringUtils 类与 String 类的区别在于：此类是 null 安全的，即如果输入参数 String 为 null，则不会抛出NullPointerException 异常，代码更健壮。
以函数 isEmpty 为例子：
存在字符串stringTest， 若该字符串为空，返回 true

1.使用String类判断方法为：

```java
if (null != stringTest) {
	if (stringTest.isEmpty()) {
		return true;
	}
} else｛
	return true;
｝
```

该方法需要先进行非空判断，已避免空指针。

2.使用StringUtils的判断方法为：

```java
if (StringUtils.isEmpty(stringTest)) {
	return true;
}
```

查看 StringUtils 的源码可知：

```java
public static boolean isEmpty(String str) {
	return (str == null) || (str.length() == 0);
 }
```

其相关的操作已经处理好。再查看常见操作trim 函数的源码如下：

```java
public static String trim (String str) {
    return str == null ? null : str.trim();
}
```

因此可知，使用StringUtils类比原始的String类更加健壮，避免空指针。
常见方法如下

# 1. 判断函数

1. 判断是否为空
   `StringUtils.isEmpty(String str)`

2. 判断是否非空
   `StringUtils.isNotEmpty(String str)`

3. 判断空白
   `StringUtils.isBlank(String str)`
4. 判断非空白
   `StringUtils.isNotBlank(String str)`
5. 判断是否存在空白(数组)
   `StringUtils.isAnyBlank(CharSequence… css)`

6. 判断是否存在空(数组)
   `StringUtils.isAnyEmpty(CharSequence… css)`

7. 判断不存在空白(数组)
   `StringUtils.isNoneBlank(CharSequence… css)`

8. 判断不存在空(数组)
   `StringUtils.isNoneEmpty(CharSequence… css)`

9. 判断是否空白

   `StringUtils.isWhitespace(CharSequence cs)`

# 2. 大小写函数

1. 首字母大写
   `StringUtils.capitalize(String str)`
2. 首字母小写
   `StringUtils.uncapitalize(String str)`
3. 全部大写
   `StringUtils.upperCase(String str)`
4. 全部小写
   `StringUtils.lowerCase(String str)`
5. 大小写互相转化
   `StringUtils.swapCase(String str)`
6. 判断是否全大写
   `StringUtils.isAllUpperCase(CharSequence cs)`
7. 判断是否全小写
   `StringUtils.isAllLowerCase(CharSequence cs)`

# 3. 删除函数

1. 从字符串中删除某字符
   `StringUtils.remove(String str, char remove)`
2. 从字符串中删除字符串
   `StringUtils.remove(String str, String remove)`
3. 删除结尾匹配的字符串
   `StringUtils.removeEnd(String str, String remove)`
4. 删除结尾匹配的字符串，忽略大小写，返回String：
   `StringUtils.removeEndIgnoreCase(String str, String remove)`
5. 正则表达式删除字符串
   `StringUtils.removePattern(String source, String regex)`

6. 删除开头匹配的字符串
   `StringUtils.removeStart(String str, String remove)`

7. 删除开头匹配的字符串，忽略大小写

   `StringUtils.removeStartIgnoreCase(String str, String remove)`

8. 删除所有空格，包括中间
   `StringUtils.deleteWhitespace(String str)`

# 4. 字符替换函数

1. 用 replacement 替换 searchString 字符串
   max 表示替换个数，默认全替换，为 -1，可不填。0表示不换。其他表示从头开始替换 n 个

   `StringUtils.replace(String text, String searchString, String replacement, int max)`

2. 仅替换一个，从头开始
   `StringUtils.replaceOnce(String text, String searchString, String replacement)`

3. 多个替换, searchList 与 replacementList 需一一对应
   `StringUtils.replaceEach(String text, String[] searchList, String[] replacementList)`

4. 多个循环替换,searchList 与 replacementList 需一一对应

   `StringUtils.replaceEachRepeatedly(String text, String[] searchList, String[] replacementList)`

5. 替换 start 到 end 的字符，返回String：
   `StringUtils.overlay(String str, String overlay, int start, int end)`

# 5. 拆分合并函数

1. 特定符号分割字符串，默认为空格，可不填
   `StringUtils.split(String str)`

2. 特定符合分割字符串为长度为 n 的字符数组，n为 0，表示全拆

   `StringUtils.split(String str, String separatorChars, int n)`

3. 合并函数，数组合并为字符串
   `StringUtils.join(byte[] array,char separator)`

4. 合并函数，separator 为合并字符，当为null时，表示简单合并，亦可不填；startIndex和endIndex表示合并数组该下标间的字符，使用separator字符，亦可不填，表示全合并
   `StringUtils.join(Object[] array,char separator,int startIndex,int endIndex)`

# 6. 截取函数

1. 截取字符串
   `StringUtils.substring(String str,int start)`
2. 从某字符后字符开始截取
   `StringUtils.substringAfter(String str,String separator)`
3. 截取至最后一处该字符出现
   `StringUtils.substringBeforeLast(String str,String separator)`
4. 从第一次该字符出现后截取
   `StringUtils.substringAfterLast(String str,String separator)`
5. 截取某字符中间的子字符串
   `StringUtils.substringBetween(String str,String tag)`

# 7. 删除空白函数

1. 删除空格
   `StringUtils.trim(String str)`
2. 转换空格为empty
   `StringUtils.trimToEmpty(String str)`
3. 转换空格为 null
   `StringUtils.trimToNull(String str)`
4. 删除所有空格，包括字符串中间空格
   `StringUtils.deleteWhitespace(String str)`

# 8. 判断是否相等函数

1. 判断是否相等
   `StringUtils.equals(CharSequence cs1,CharSequence cs2)`
2. 判断是否相等，忽略大小写
   `StringUtils.equalsIgnoreCase(CharSequence cs1,CharSequence cs2)`

# 9. 是否包含函数

1. 判断第一个参数字符串，是否都出现在参数2中
   `StringUtils.containsOnly(CharSequence cs,char… valid)`

2. 判断字符串中所有字符，都不在参数2中
   `StringUtils.containsNone(CharSequence cs,char… searchChars)`

3. 判断字符串是否以第二个参数开始
   `StringUtils.startsWith(CharSequence str,CharSequ～ence prefix)`

4. 判断字符串是否以第二个参数开始，忽略大小写

   `StringUtils.startsWithIgnoreCase(CharSequence str,CharSequence prefix)`

