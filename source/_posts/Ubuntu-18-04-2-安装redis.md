---
title: Ubuntu 18.04.2 安装redis
date: 2019-07-28 22:49:37
categories: Redis学习笔记
tags:
- Redis

---

# 1. 前言

Redis 是目前业界使用最广泛的内存数据存储。相比 Memcached，Redis 支持更丰富的数据结构，例如 hashes, lists, sets 等，同时支持数据持久化。除此之外，Redis 还提供一些类数据库的特性，比如事务，HA，主从库。可以说 Redis 兼具了缓存系统和数据库的一些特性，因此有着丰富的应用场景。本文介绍 Redis 在Ubuntu 18.04.2 系统下的安装，为后续学习奠定基础。

[redis官网](https://redis.io/)

<!--more-->

# 2. 安装

## 2.1 安装 Redis 服务器端

```sh
$ sudo apt-get update
$ sudo apt-get install redis-server
```

安装完 redis-server 后，redis-server 服务会自动开启，我们查看 redis-server 信息

```sh
$ ps -aux|grep redis
redis    29179  0.1  0.3  51664  3856 ?        Ssl  14:59   0:00 /usr/bin/redis-server 127.0.0.1:6379
```

可以看到 redis-server 服务已经开启，已经监听了本机的 6379 端口号

## 2.2 通过命令行客户端访问 Redis

安装 Redis 服务器，会自动地一起安装 Redis 命令行客户端程序。

在本机输入 redis-cli 命令就可以启动，客户端程序访问 Redis 服务器。

```sh
$ redis-cli
127.0.0.1:6379> ping
PONG
```

当我们输入了 ping，能出现 PONG 就说明 redis 已经成功安装。

# 3. 配置

redis 的配置文件位置为 `/etc/redis/redis.conf` ，下面例举几个重要配置

```sh
# 配置能访问该redis服务的ip，注释即为允许所有ip访问（这里我注释掉了）
# bind 127.0.0.1 ::1
# redis服务端口号，默认即为6379
port 6379
# 是否在后台执行，yes：后台运行；no：不是后台运行
daemonize yes
# 数据库的数量，默认使用的数据库是DB 0。可以通过”SELECT “命令选择一个db
databases 16
# 把数据库存到磁盘上: 
# save <seconds> <changes> 
# 会在指定秒数和数据变化次数之后把数据库写到磁盘上。 
# 
# 下面的例子将会进行把数据写入磁盘的操作: 
# 900秒（15分钟）之后，且至少1次变更 
# 300秒（5分钟）之后，且至少10次变更 
# 60秒之后，且至少10000次变更 
# 
# 注意：你要想不写磁盘的话就把所有 "save" 设置注释掉就行了
save 900 1 
save 300 10 
save 60 10000
# 登录redis服务时的认证密码，默认为foobared（这里我更改为了123456）
requirepass 123456 
```

修改完配置文件后，重启 redis 服务

```sh
$ sudo /etc/init.d/redis-server restart
[ ok ] Restarting redis-server (via systemctl): redis-server.service.
```

此时再通过 redis-cli 登录 redis 服务就需要指定登录密码

```sh
$ redis-cli -a 123456
127.0.0.1:6379> ping
PONG
# 未认证登录
$ redis-cli
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> ping
PONG
```

同时我们也可以远程访问该 redis 服务

```sh
redis-cli -h yourhost -a 123456
```

将 `yourhost` 更改为启动 redis 服务的机器的 ip 地址，即可远程访问 redis 服务

# 4. 参考资料

1. [Ubuntu16.04安装Redis](https://www.cnblogs.com/zongfa/p/7808807.html)