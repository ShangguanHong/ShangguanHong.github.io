---
title: "从Hexo迁移到Hugo"
date: 2020-03-20T22:26:01+08:00
lastmod: 2020-03-20T22:26:01+08:00
draft: false
categories: [工具使用]
tags: [hugo, hexo]
---

之前使用Hexo工具来作为博客生成工具的，使用了大概半年左右，期间换过了一两次电脑，这就使得每次使用的时候都需要重新下载 nodejs、npm，然后下载安装hexo，执行 `npm install` 等等一系列的命令，期间可能还会遇到各种各样的问题。最近看到了Hugo这款博客生成工具，它是使用Go语言编写的，只有一个二进制文件，使用非常简单，在尝试了之后决定将之前在hexo的博文全部迁移到hugo上。

## hugo的优点

- 使用go语言编写，运行效率非常快，使用hexo生成网页大约需要5s，使用hugo只需要1000ms即可完成.
- 使用方便，只有一个二进制文件，完全不需要依赖第三方的工具.
- 后续更换电脑时也方便，只需要在新环境上重新下载一下hugo即可继续编写博文.

更多的关于Hugo的介绍，请参考Hugo的官网 https://gohugo.io/ 。

## 安装hugo

### 手动下载

我们可以直接从[Github Release](https://github.com/gohugoio/hugo/releases)页面下载对应的二进制文件,然后把它放在你的PATH目录里即可使用。这个支持任何平台，根据自己的平台选择相应的二进制包即可，最后记得将hugo可执行文件的位置加入到对应的环境变量中。

### 命令行下载

```sh
# Mac
$ brew install hugo
# ubuntu or debian
$ sudo apt-get install hugo
# Windows 
choco install hugo -confirm
```

安装完成后，可以运行如下命令查看其版本号，如果成功输出则说明安装成功

```sh
$ hugo version
Hugo Static Site Generator v0.67.1-4F44227B windows/amd64 BuildDate: 2020-03-15T19:32:32Z
```

## 使用hugo

### 创建站点

在空闲目录下，建立一些名为`blog`的站点，使用下面 的命令：

```bash
$ hugo new site blog
```

在blog目录下，会出现 `archetypes/ config.toml content/ data/ layouts/ static/ themes/`文件或文件夹，`config.toml` 就是博客的配置文件，`archetypes` 目录下有一个 `default.md`，存放 的是默认的建立新博文时候使用的模板，可以根据自己需求修改。`content/` 目录用来存 放博文，`static/` 可以存放一些自己的文件，`theme/` 文件夹用于存放不同的主题。

### 安装主题

建立站点以后，我们需要安装主题，Hugo 提供了一个[主题浏览网站 ](https://themes.gohugo.io/)，可以浏览各种主题的效果。这里我选择了 [even](https://github.com/olOwOlo/hugo-theme-even) 主题作为本站的使用主题。

在博客根目录下，使用下面命令安装主题：

```bash
$ git init
$ git submodule add https://github.com/olOwOlo/hugo-theme-even.git theme/even
```

然后编辑 `config.toml`，设置主题为 even

```toml
theme = "even"
```

更多该主题的使用说明，请查看该主题的 [中文说明](https://github.com/olOwOlo/hugo-theme-even/blob/master/README-zh.md)

### 新建博文

假如要建立新的博文 `test.md`，使用下面的命令

```bash
$ hugo new post/test.md
```

这个命令会在 `content` 目录下建立 `post` 目录，并在 `post` 下生成 `test.md` 文 件，博文书写就在这个文件里使用 Markdown 语法完成。博文的 front matter 里 `draft` 选项默认为 `true`，需要改为 `false` 才能发表博文，建议直接更改上面说的 `archetypes` 目录下的 `default` 文件，把 `draft: true` 改为 `draft: false`，这 样生成的博文就是默认可以发表的。

和 Hexo 一样，在 front matter 里面，使用 `tags` 指定文章的 tag，使用 `categories` 指定文章的类别，示例如下：

```yaml
tags: [LaTeX, Hexo, font]
categories: [Mac, Linux]
```

可以把 Hexo 博客里的所有博文 Markdown 文件拷贝到 `content/post/` 目录下

### 查看效果

为了查看生成的博客的效果，可以使用 `hexo server` 命令，示例输出如下

```sh
$ hugo server
Building sites …
                   | ZH-CN
-------------------+--------
  Pages            |   183
  Paginator pages  |     7
  Non-page files   |     0
  Static files     |    62
  Processed images |     0
  Aliases          |    60
  Sitemaps         |     1
  Cleaned          |     0

Built in 736 ms
Watching for changes in D:\MyBlogs\{archetypes,content,data,layouts,static,themes}
Watching for config changes in D:\MyBlogs\config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

该命令会生成的一个本地的博客服务器，直接访问网址 http://localhost:1313/ 即可查看生成的博客效果。

## 部署到github

本部分假定你在 github pages 建立的是 *User/Organization Pages* ，地址名称形式为 `https://<USERNAME|ORGANIZATION>.github.io/`，如果不是这种类型地址，请参考[官方 指导](https://gohugo.io/hosting-and-deployment/hosting-on-github/)。

1. 先创建一个 `.gitignore` 文件，内容如下

```toml
public/*
themes/*
resources/*
```

2. 运行下列命令，期间可能输入github账号密码

```bash
$ git add .
$ git commit -m "init commit"
$ git remote add origin "你的仓库地址"
$ git push origin hugo
```

这样即可将本地文件全部上传到 hugo 分支，接着按照[使用Travis CI自动部署hugo](https://shangguanhong.github.io/2020/03/20/使用travis-ci自动部署hugo/)文章的操作，即可自动部署网页，或者参照[官方文档](https://gohugo.io/hosting-and-deployment/hosting-on-github/)的做法自己写一个脚本文件也可以。

## 遇到的问题

### 文章的permalinks设置

Hexo 中，文章的链接都是 `YYYY/MM/DD/post_name/` 的格式，在 Hugo 中，可以通过设 置 `permalinks` 参数来实现平滑过渡，在 `config.toml` 中加入以下设置：

```toml
[permalinks]
  post = "/:year/:month/:day/:filename/"
```

### 大小写问题

[Hugo 默认会把 url 中的大写字符转为小写 ](https://discourse.gohugo.io/t/disable-hugo-case-sensitive-url-matching/2498/4)，如果不想转换，在配置文件中加入

```toml
disablePathToLower = true
```

Hugo 默认会把分类（`categories`）名称转为小写，如果不想转换，在配置中加入，

```toml
preserveTaxonomyNames = true
```

### tags和categories

与hexo不同, hugo使用的是类似于数组的形式的, 例如本篇的tags就是`["hexo", "hugo"]`, 而在hexo中则是

```javascript
Hexo中配置
tags:
 - hexo
 - hugo

Hugo中配置
tags: ["hexo", "hugo"]
```

### 图片的上传

这里建议使用github图床+PicGo工具的使用组合来解决图片问题，具体参考该文章 [*PicGo*+*GitHub图床*,让Markdown飞 ](http://www.baidu.com/link?url=JBkKFLYUzTqPzHeyNk-NG91i9bkIAUFK1IG9KDnjKvSB5_EkSwoQAkaQZGMluJY7)