---
title: "github使用Travis CI"
date: 2019-12-19 20:39:10
categories: ["工具使用"]
tags: ["hexo", "github", "Travis CI"]
---

# 前言

使用 hexo + gitPages 搭建个人博客的都知道，每当要发表一篇博文的时候，都需要手动使用 `hexo g`  生成静态网页，然后再通过 `hexo d` 命令将静态文件推送到 GitHub远程仓库，这段操作在你执行一次两次的时候还不觉得有什么，但是当你频繁的修改博文的时候，你就会明白每次都需要执行这两条语句是多么的枯燥。对于一个程序猿来说，最不想干的就是一些重复且枯燥的工作（懒惰促使程序猿进步！）😋。现在，我们可以通过 Travis CI自动构建博客，我们只需要将写好的博文推送到GitHub上即可。

# Travis CI介绍

[Travis CI](https://travis-ci.org/) 是目前新兴的开源持续集成构建项目，它与 jenkins，GO的很明显的特别在于采用 yaml 格式，简洁清新独树一帜。目前大多数的 github 项目都已经移入到 Travis CI 的构建队列中，据说 Travis CI 每天运行超过 4000 次完整构建

# Hexo介绍

参考之前的文章 : [利用GithubPages + Hexo搭建自己的博客](https://shangguanhong.github.io/2019/05/28/GithubPages + Hexo搭建自己的博客/)

# 使用Travis自动构建

这里假设使用 hexo + githubpages 搭建好了自己的博客，并且 hexo 的源码放在了hexo分支上。

如果你并不理解这是什么意思，请移步至我之前写的关于 hexo 的文章查看。

## 获取 Access Token

接下来我们首先需要在 github 上拿到一个 Access Token（访问令牌），获取方式如下：

1. 打开个人设置 -> 点击最下侧的 `Developer settings` 选项卡 -> 选择 `Perrsonal access tokens` -> 点击 `Generate new token`
2. 填入 Note，这里我填的是 TravisCI，然后勾选以下红色框中的权限即可

![image-20191219211050443](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20191219211050443.png)

3. 点击 `Generate token`，即可生成 access token，将此token复制保存好，后续步骤需要用到。

## 配置Travis CI

如果之前从未使用 [Travis CI](https://travis-ci.org/) 来构建项目，则我们先需要使用GitHub账号来登录网站,登录进来后，会进到如下图界面，如果底下 没有把 GitHub 仓库中的项目加载进来，可以手动点击左上角的 `Sync account` 按钮，待到同步完成后将要自动构建的项目开启

![image-20191219211426850](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20191219211426850.png)

点击1号按钮，添加需要自动构建的仓库，然后点击2号按钮进行设置

进入设置界面，在 `Environment Variables` 处填入之前获取的 `Access Token`,  `Name` 填入 `GH_TOKEN`

![image-20191219211746069](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20191219211746069.png)

## 配置.travis.yml

在项目根目录下创建 `.travis.yml` 文件，Travis CI 在自动构建时从 .travis.yml 文件中获取这些配置信息，以此来完成构建任务；配置信息主要包括源码分支，静态文件推送分支，仓库地址等信息。

文件内容如下：

```yml
language: node_js
node_js: stable

# .travis.yml 配置
# 指定缓存模块，可选。缓存可加快编译速度。
cache:
  directories:
    - node_modules

# S: Build Lifecycle
install:
  - npm install

#before_script:
 # - npm install -g gulp

# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo g

# 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
after_script:
  - git config user.name "ShangguanHong"
  - git config user.email "sgh1450280694@gmail.com"
  # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
  - sed -i "s/gh_token/${GH_TOKEN}/g" ./_config.yml
  - hexo d
# E: Build LifeCycle

branches:
  only:
    - hexo
```

修改  `_config.yml` 文件的deploy节点：

原内容：

```yml
deploy:
  type: git
  repo: git@github.com/ShangguanHong/ShangguanHong.github.io.git
  branch: master
```

新内容：

```
deploy:
  type: git
  repo: https://gh_token@github.com/ShangguanHong/ShangguanHong.github.io.git
  branch: master
```

至此，配置已经结束。

这时你需要将本次的修改提交到 github 的 hexo 分支，然后回到 Travis 页面，就可以看到 Travis 正在执行你写在 `.travis.yml` 中的脚本

![image-20191219212754905](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20191219212754905.png)

如果你修改了文章，那么刷新博客界面也会发现已经修改了，从此再也不用手动 `hexo g` 和 `hexo d` 了！！

观察上图右上角的徽章，是不是觉得挺眼熟的？没错，在大型项目中的 readme 中基本都会有它的身影，那么如何将它放入 readme 中呢？这个操作十分的简单，点击该图标，将 `FORMAT` 选择成 `MarkDown` 即可，复制下面的 `RESULT` ，粘贴到你的 readme 文件中即可。

![image-20191219213055541](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20191219213055541.png)



