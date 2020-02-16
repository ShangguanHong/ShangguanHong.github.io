---
title: Ubuntu18.04更换国内源
toc_number: true
copyright: true
date: 2020-02-16 21:06:13
categories:
- Linux学习笔记
tags:
- linux
- ubuntu
---

1. 备份源文件，以防止操作失误后无法还原文件

   ``` sh
   sudo cp /etc/apt/sources.list /ect/apt/sources.list.backup
   ```

2. 打开 `/etc/apt/sources.list ` 文件

   ```sh
   sudo vim /ect/apt/sources.list
   ```

3. 将源文件里的内容全部注释或者全部删除，然后写入以下内容

   - 阿里云源

     ```
     deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
     deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
     deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
     ```

   - 清华源

     ```
     deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
     deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
     deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
     deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
     deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
     deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
     deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
     deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
     deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
     deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
     ```

   - 中科大源

     ```
     deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
     deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
     deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
     deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
     deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
     deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
     deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
     deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
     deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
     deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
     ```

   4. 保存后执行以下命令

      ```sh
      # 更新源列表
      sudo apt update
      # 更新软件
      sudo apt upgrade
      ```

      

