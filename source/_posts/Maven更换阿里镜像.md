---
title: Maven更换阿里镜像
date: 2019-08-07 19:12:57
copyright: true
categories: 更换镜像
tags:
- Maven
---

# 1. 前言

`Maven` 默认的中央仓库速度慢，可以考虑换成阿里的镜像。修改方式主要有两种，全局修改和针对当前项目的修改。

# 2. 全局修改

## 2.1 Windows

`Windows` 下 `maven` 的 `.m2` 文件夹地址在 `C:\User\****\.m2` (***为具体的用户名)

在该路径下新建一个 `setting.xml` 文件(如果不存在的话)

内容如下:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">

      <mirrors>
        <mirror>  
            <id>alimaven</id>  
            <name>aliyun maven</name>  
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
            <mirrorOf>central</mirrorOf>          
        </mirror>  
      </mirrors>
</settings>
```

<!--more-->

## 2.2 Linux

`Linux` 下 `maven` 的 `.m2` 文件夹地址为 `~/.m2` , `~` 为用户根目录

接下来操作与 `Windows` 一样

# 3. 针对当前项目修改

在项目的 `pom.xml` 文件内的 `<repositories>` 节点下面增加新节点

```xml
<repository>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</repository>
```