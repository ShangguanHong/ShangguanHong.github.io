---
title: "使用 GitHub Actions + Hugo 自动发布博文"
date: 2022-06-26T00:49:46+08:00
lastmod: 2022-06-26T00:49:46+08:00
draft: false
categories: [工具使用]
tags: ["GitHub Actions", "Hugo"]
---



之前写过一篇 [使用 Travis CI 自动部署 Hugo](https://shangguanhong.github.io/2020/03/20/%E4%BD%BF%E7%94%A8travis-ci%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2hugo/) 的文章，最近更新文章的时候发现失效了。查看失败原因，原来是因为之前使用的 travis-ci 的网站已经更新了，变成了使用 travis-ci.com，因此需要上 travis-ci.com 上重新配置一次。

正好之前有了解过 GitHub 官方自己就出了一个持续集成的工具--GitHub Actions，借着这次的机会就直接将自动发布博文的工作交给 GitHub Actions 好了。

有对 GitHub Actions 还不了解的，可以先看下我之前写的 [GitHub Actions介绍](https://shangguanhong.github.io/2022/06/25/github-actions%E4%BB%8B%E7%BB%8D/) 这篇文章，对 GitHub Actions 先有一个基本的了解。

**说明**：

- 使用的 hugo 源文件存放在 hugo 分支下
- 网站的静态文件存放在 master 分支下



在 `.github/workflows` 目录下新建一个 `main.yaml` 文件，内容如下：

```yaml
name: deploy

on:
  push: # push的时候触发
    branches: # 那些分支需要触发
      - hugo

jobs:
    build:
        runs-on: ubuntu-20.04
        steps:
        	# 同步 .gitmodules 文件里的模块
            - name: Checkout
              uses: actions/checkout@v2
              with:
                  submodules: true
                  fetch-depth: 0
			# 安装 hugo 
            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "latest"
			# 编译静态页面文件
            - name: Build Web
              run: hugo
			# 推送静态页面到 master 分支
            - name: Deploy Web
              uses: peaceiris/actions-gh-pages@v3
              # 详细配置如下：https://github.com/peaceiris/actions-gh-pages#options
              with:
              	  # 同一个仓库的话直接这样配置就好了
                  github_token: ${{ secrets.GITHUB_TOKEN }}
                  # 推送到 master 分支
                  PUBLISH_BRANCH: master
                  # 推送 ./public 文件夹下的文件
                  PUBLISH_DIR: ./public
                  commit_message: ${{ github.event.head_commit.message }}
```

将上述文件提交到 GitHub 上后，点击仓库上方的 Actions 图标即可看到 GitHub Actions 正在执行上述文件中定义的操作。



![image-20220626013105068](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/img/image-20220626013105068.png)



最终，当 `pages build and deployment` 执行完毕之后，即可刷新博客网站，就可以看到最新的内容已经自动更新了。
