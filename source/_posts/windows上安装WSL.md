---
title: windows上安装WSL
toc_number: true
copyright: true
date: 2019-11-21 20:12:53
categories: 
- Linux学习笔记
tags:
- linux
- wsl
---

# 介绍

Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。

意思就是一个能在windows系统上运行的linux系统。

下面我以介绍在 windows 10 中安装 ubuntu 18.04 LTS为例子，来说明如何进行安装。

<!--more-->

# 安装

1. 首先在控制面板->程序->程序与功能->启用或关闭windows功能界面中，勾选**适用于Linux的Windows子系统**，然后重启电脑

   ![image-20191121201819830](windows%E4%B8%8A%E5%AE%89%E8%A3%85WSL/image-20191121201819830.png)

2. 在 `Microsoft store` 中搜索 ubuntu，会出现许多windows支持的linux系统，这里我选择 ubuntu 18.04 LTS版本

   ![image-20191121202316844](windows%E4%B8%8A%E5%AE%89%E8%A3%85WSL/image-20191121202316844.png)

3. 安装完成后在开始菜单上就会显示一个 Ubuntu 的图标，这就表示该Ubuntu已经安装完成，点击它设置初始密码，就可以将其当作一个 Ubuntu 系统来使用了。

# 后续

## 美化

1. 更换配色

下载 microsoft 官方出品的终端调色工具，ColorTool

 github源码地址：[https://github.com/Microsoft/Terminal/tree/master/src/tools/ColorTool](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMicrosoft%2FTerminal%2Ftree%2Fmaster%2Fsrc%2Ftools%2FColorTool) 

github下载地址： https://github.com/microsoft/terminal/releases/tag/1904.29002 

下载完后解压压缩包，管理员身份运行cmd到压缩目录下，运行 `ColorTool.exe -d schemes/solarized_dark.itermcolors` 命令，显示 Wrote selected scheme to the defaults. 表示调色成功，重启 wsl 生效。

2. 更换字体

Fire Code Retina 字体的下载地址为： [https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/ttf/FiraCode-Retina.ttf](https://links.jianshu.com/go?to=https%3A%2F%2Fraw.githubusercontent.com%2Ftonsky%2FFiraCode%2Fmaster%2Fdistr%2Fttf%2FFiraCode-Retina.ttf) 

下载完后双击，安装即可。

然后在 wsl 终端，右键终端上方的空白处点击属性，将字体更改为 Fire Code Retina 即可，这时回去看终端字体，是不是感觉心旷神怡

![image-20191121203951851](windows%E4%B8%8A%E5%AE%89%E8%A3%85WSL/image-20191121203951851.png)

![image-20191121204047086](windows%E4%B8%8A%E5%AE%89%E8%A3%85WSL/image-20191121204047086.png)

## 换源与ZSH

参考：[Linux美化](https://shangguanhong.github.io/2019/11/21/Linux美化/)

