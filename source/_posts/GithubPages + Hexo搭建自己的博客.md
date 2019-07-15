---
title: 利用GithubPages + Hexo搭建自己的博客
date: 2019-05-28 20:48:13
categories: Hexo学习笔记
tags:
- GithubPages
- Hexo
---



## 1. 部署本地博客

1. 安装Node.js, 参考 <https://www.runoob.com/nodejs/nodejs-install-setup.html>

<!--more-->

1. 安装Git, 参考 <https://www.runoob.com/git/git-install-setup.html>

2. 安装Hexo：右键选择Git Bash，输入命令 `npm install -g hexo-cli` 

   安装完成后，输入 `hexo -v` 查看是否安装成功 

   > 下列所有的命令都是在Git Bash内运行

3. 在合适的位置创建站点目录, 进入站点目录依次运行以下命令

```markdown
hexo init
npm install
```

​		成功运行后，站点目录下会产生新文件和目录

5. 启动服务器：在站点目录下运行 `hexo s` 启动服务, 在浏览器输入 http://localhost:4000/ 便可看到本地博客已经部署完成

## 2. 绑定GithubPages

1. 如果没有Github账号的创建Github账号, 有的直接登录即可

2. 点击右上角+号，选择New repository

3. 创建一个和你用户名相同的仓库，后面加.github.io，例如xxxx.github.io，其中xxx就是你注册GitHub的用户名，点击create repository

4. 生成ssh添加到github

   4.1 输入命令 `ssh-keygen -t rsa -C "youreamil"` ，连续回车三下，此时在你的电脑用户目录下会有一个隐藏文件夹.ssh，打开它能发现有id_rsa和id_rsa.pub两个文件

   4.2  在github的setting中, 打开SSH keys设置选项，点击New SSH key，将id_rsa.pub中的内容拷贝进去

   4.3 输入命令 `ssh -T git@github.com` ，查看ssh是否配置成功, 期间可能会让你输入yes, 如果显示了successfully即配置成功

5. 将本地Hexo博客部署到GithubPages

   5.1 安装 `hexo-deployer-git` 插件。输入命令 `npm install hexo-deployer-git --save` 
   
   5.2 修改站点目录下的_config.yml文件
   
   ```
   # Deployment
   ## Docs: https://hexo.io/docs/deployment.html
   deploy:
     type: git
     repo: git@github.com:<github账号名称>/<github账号名称>.github.io.git
     branch: master
   ```
   
   5.3 输入命令 `hexo g && hexo d` 将本地Hexo博客推送到GithubPages
   
6. 等一会儿, 浏览器输入：`https://<github账号名称>.github.io` ,即可看到本地博客推送到了GithubPages上，之后每次修改本地博客的内容需要推送到GithubPages上时都需要执行 `hexo g && hexo d` 命令



