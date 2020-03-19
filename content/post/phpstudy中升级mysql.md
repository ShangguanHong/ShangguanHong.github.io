---
title: phpstudy中升级mysql
date: 2019-05-28 14:45:56
copyright: true
categories: [MySQL学习笔记]
tags: [PHPstudy, MySQL]
---

## 1. 正文

1. 从官网下载对应平台最新的 MySQL 解压缩版 <http://www.mysql.com/downloads/>

   <!-- more -->

2. 解压缩下载的文件，复制到phpStudy的MySQL文件夹下，我是把文件夹清空后复制过去的；

3. 将../phpStudy/MySQL/bin路径追加到Path

4. 复制一份my-default.ini为my.ini；

5. 配置my.ini，修改下面两处即可，别忘记去掉前面的#

   `basedir=../MySQL/MySQL(mysql所在目录)`

   `datadir=../MySQL/MySQL/data(mysql所在目录/data)`

6. 以管理员运行cmd，cd到../phpStudy/MySQL/bin,先执行 

   `mysqld -remove`, 

   再执行

   `mysqld -install`

7. 然后就可以用net start mysql来启动mysql，用net stop mysql来停止mysql了。用phpStudy中的按键也可以。

## 2. 附录
### 1. phpStudy中升级MySQL版本详细介绍
### http://www.jb51.net/article/120263.htm
### 如何解决更新后phpStudy开启不了MySQL服务的问题
### https://www.cnblogs.com/mikusnail/p/8422013.html