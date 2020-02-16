---
title: Linux美化
toc_number: true
copyright: true
date: 2019-11-21 21:11:12
categories:
- Linux学习笔记
tags:
- linux
- zsh
---

记录每次安装Linux系统必完成的流程，方便以后的开发

<!--more-->

# zsh

```sh
# ubuntu
$ sudo apt-get install zsh
# contos
$ sudo yum install zsh -y
# arch
$ sudo pacman -S zsh
```



# on-my-zsh

## 安装

### 通过在线脚本安装

```sh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## 主题

```sh
$ vim ~/.zshrc
```

找到 `ZSH_THEME`，将其更改为 `ZSH_THEME="agnoster"`

```sh
# 更新配置
$ source ~/.zshrc
```

## 插件

```sh
# 语法高亮 
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
# 自动补全
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
# 自动跳转 j指令跳转
sudo pacman -S autojump
# 修改配置文件
vim ~/.zshrc
# 在plugins后括号里添加安装的插件名字
# extract 自带的解压缩工具，使用x指令解压全部格式的压缩包
plugins=(
    git
    autojump
    extract
    zsh-syntax-highlighting
    zsh-autosuggestions
)
# 更新配置文件
source ~/.zshrc    
```

# 换源

## ubuntu 18.04

1. 首先进行备份

```sh
$ sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 
```

2. 更改 sources.list 文件内的内容

```sh
# 打开sources.list
$ sudo vim /etc/apt/souces.list
```

3. 删除 sources.list 文件的内容(ggdG)，并且填入以下内容

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```

4. 更新

```sh
# 更新软件列表
$ sudo apt-get update
# 更新软件
$ sudo apt-get upgrade
```

## Manjaro

manjaro 使用的包管理工具是 pacman，它的源配置文件在 `/etc/pacman.d/mirrorlist`文件中，如果没有可以自己创建一个，一般都是有的，然后将原来的源删除，添加自己需要的即可，如果你不知道你需要换的源 url 的话可以使用下面的方法。

输入以下指令，会自动寻找最快的源，只需要勾选即可

```sh
$ sudo pacman-mirrors -i -c China -m rank
```

  ![深度截图plasmashell_20190901203111](https://shangguanhong.github.io/2019/09/01/manjaro%E5%AE%89%E8%A3%85%E5%AE%8C%E5%90%8E%E9%9C%80%E8%A6%81%E5%81%9A%E7%9A%84%E4%BA%8B/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE_plasmashell_20190901203111.png)

 这里我选择的是 华中科技大学(USTC)的镜像源，校外人员可以使用华为的。

在执行完上面的操作之后，在 `/etc/pacman.conf` 文件末尾添加两行：

```conf
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

然后请安装 `archlinuxcn-keyring` 包以导入 GPG key。

安装 yay

```sh
# aur社区上的软件需要使用yay来下载
$ sudo pacman -S yay
# 打开pacman与yay的color
$ sudo vim /etc/pacman.conf
# 找到color将其取消注释
```

在换源之后，就可以使用 `sudo pacman -Syyu` 更新一下软件。