---
title: Windows上使用docker遇到的问题(持续更新...)
toc_number: true
copyright: true
date: 2019-11-21 22:21:50
categories:
- Docker学习笔记
tags:
- docker
---

这里记录了一些在windows系统上使用docker的时候遇到的一些问题，并给出最终解决方法。

<!--more-->

# 在WSL中使用docker

## 使用DOCKER_HOST

[在Linux的Windows子系统上(WSL)使用Docker（Ubuntu）](https://www.cnblogs.com/xiaoliangge/p/9134585.html)

## 使用WLS2

### 介绍

WSL 2是Windows中Linux子系统的新版本，以便在Windows上运行ELF64 Linux二进制文件。它使用了真正的Linux内核的新架构，改变了Linux二进制文件与Windows和计算机硬件的交互方式，但仍提供与WSL 1（当前广泛使用的版本）相同的用户体验。WSL 2提供了更快的文件系统性能和完整的系统调用兼容性，使你可以运行更多像Docker这样的应用程序。 

### 要求

Windows系统版本要大于等于 18917 ，可以在 “运行”里面输入 `winver` 查看当前Windows系统版本

![image-20191123103554851](Windows%E4%B8%8A%E4%BD%BF%E7%94%A8docker%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/image-20191123103554851.png)

### 安装

1. **管理员身份** 运行 Windows PowerShell，输入 `Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform`

   ![image-20191123103818134](Windows%E4%B8%8A%E4%BD%BF%E7%94%A8docker%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/image-20191123103818134.png)

​			按需求看是否需要重启，我这里是因为之前已经安装过了所以不需要，如果是第一次安装的需要重启电脑

2. 将 WSL 的默认版本设置为2，输入 `wsl --set-default-version 2`

   ![image-20191123104352863](Windows%E4%B8%8A%E4%BD%BF%E7%94%A8docker%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/image-20191123104352863.png)

3. 如果之前没安装过 Linux 子系统的直接去安装默认就已经使用WSL2了，如果之前安装过了请接着看如何调整

4. 查看所有已经安装的子系统于其正在使用的wsl版本，输入 `wsl -l -v`

   ​	![image-20191123104804372](Windows%E4%B8%8A%E4%BD%BF%E7%94%A8docker%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98/image-20191123104804372.png)

5. 我这里是已经做过了转换所以版本已经是2了，如果显示是1的，输入 `wsl  --set-version Ubuntu-18.04 2` 等待转换即可

6. 完成转换之后，就可以进入子系统里面安装并且使用docker了

# WSL内修改docker源

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["你的加速地址"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

然而当执行到 `sudo systemctl daemon-reload` 的时候系统会报错，提示 `System has not been booted with systemd as init system (PID 1). Can't operate.)`

将下面指令更改为 `sudo service docker restart`，使用 `docker info`，查看Docker信息发现镜像源已经被更改成需要的了，因此上面那句报错忽略即可，具体报错原因目前不清楚，可能和WSL的运行机制有关系吧。

# 开启了虚拟化后启动docker依旧报错

**在管理员模式下的命令提示符中输入：bcdedit /set hypervisorlaunchtype Auto，然后重启电脑，完美解决**。 

```

```