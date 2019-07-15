---
title: 解决Redis重启后密码失效问题
date: 2019-06-04 21:48:22
categories: Redis学习笔记
tags:
- Redis

---

# 1. 前言

在用命令行设置了密码之后，下次启动redis的时候密码又没了，这是因为 `config set requirepass password` 这种设置只是临时的，当服务器重启后，密码就会失效。

真正的配置密码应该是要去配置文件中配置。

<!--more-->

# 2. 配置

打开 `redis.windows.conf` 配置文件，找到一行

```yml
# requirepass foobared
```

将前面的注释去掉，并将foobared改为自己的密码

![1559659794589](解决Redis重启后密码失效问题/1559659794589.png)

然后打开 `redis.windows-service.conf` 配置文件进行一样的操作，这样无论redis加载哪个配置文件都能保证密码一致。