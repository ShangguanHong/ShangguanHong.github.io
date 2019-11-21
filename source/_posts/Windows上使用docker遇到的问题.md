---
title: Windows上使用docker遇到的问题
toc_number: true
copyright: true
date: 2019-11-21 22:21:50
categories:
- Docker学习记录
tags:
- docker
---

这里记录了一些在windows系统上使用docker的时候遇到的一些问题，并给出最终解决方法。

<!--more-->

# 在WSL中使用docker

[在Linux的Windows子系统上(WSL)使用Docker（Ubuntu）](https://www.cnblogs.com/xiaoliangge/p/9134585.html)

# 开启了虚拟化后启动docker依旧报错

**在管理员模式下的命令提示符中输入：bcdedit /set hypervisorlaunchtype Auto，然后重启电脑，完美解决**。 

持续更新...