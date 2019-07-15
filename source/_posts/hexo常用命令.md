---
title: hexo常用命令
date: 2019-05-29 12:38:52
categories: Hexo学习笔记
tags:
- Hexo

---

# 1. Hexo

```bash
$ npm install hexp -g #安装
$ npm update hexo -g #升级
$ hexo init #初始化
```

<!--more-->

# 2. 简写

```bash
$ hexo n "title" == $ hexo new "title" # 新建博客文章
$ hexo p == $ hexo publish #新建草稿draft
$ hexo g == $ hexo generate # 生成静态文件
$ hexo s == $ hexo server # 启动服务预览
$ hexo d == $ hexo deploy # 部署上服务器
```

# 3. 服务器

``` bash
$ hexo server # hexo会监视文件变动并启动更新，无需重启服务
$ hexo server -s # hexo的静态模式
$ hexp server -p 5000 # 更改服务端口
$ hexo server -i 192.168.1.1 # 自定义IP

$ hexo clean # 清楚缓存，网页出现莫名错误时可以试试
$ hexo g # 生成静态网页
$ hexo d # 部署上服务器
```

# 4. 常见错误

## 4.1 找不到git部署

错误提示：ERROR Deployer not found: git

解决方法：运行命令 `npm install hexo-deployer-git --save`