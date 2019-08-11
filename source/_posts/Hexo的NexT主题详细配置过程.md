---
title: Hexo的NexT主题详细配置过程
date: 2019-05-29 16:18:40
copyright: true
categories: Hexo学习笔记
tags:
- Hexo
- NexT

---

# 1. 前言

之前的 [利用GitHubPages + Hexo搭建自己的博客](https://shangguanhong.github.io/2019/05/28/GithubPages + Hexo搭建自己的博客/)  中，已经拥有了一个自己的博客，但是它是非常的简陋的，所以这里将介绍Hexo中最热门的NexT主题的详细配置过程，将博客变得更加美观

<!--more-->

# 2. 安装NexT主题

在网站根目录下输入以下命令

``` bash
$  git clone https://github.com/theme-next/hexo-theme-next themes/next
```

这样在当前目录下的themes文件夹中就有了Next主题

>我们将站点根目录下的 _config.yml文件称为 `站点配置文件` , 将 themes/next 文件夹内的_config.yml文件称为 `主题配置文件` 。

# 3. 启用主题

打开`站点配置文件` ，找到 `theme` ，建议用 `ctrl+f` 搜索`theme` 快速定位，修改为

```bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

# 4. 选择主题风格

打开 `主题配置文件` ，找到 `Scheme Settings` ，`Next`主题提供四种风格，分别为 `Muse` , `Mist` , `Pisces` , `Gemini` , 使用时只需将想启用的风格前面的 `#` 删除即可，我使用的是 `Gemini` 风格的

```bash
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
# scheme: Muse
# scheme: Mist
# scheme: Pisces
scheme: Gemini
```

#  5. 菜单设置

> 菜单包括：首页、归档、分类、标签、关于等等

刚开始的时候默认只有首页和归档两个，可以根据需要添加相应的菜单，打开 `主题配置文件` ，找到 `Menu Settings` , 一下为我的设置

```bash
# ---------------------------------------------------------------
# Menu Settings
# ---------------------------------------------------------------

# When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash from link value (/archives -> archives).
# Usage: `Key: /link/ || icon`
# Key is the name of menu item. If the translation for this item is available, the translated text will be loaded, otherwise the Key name will be used. Key is case-senstive.
# Value before `||` delimeter is the target link.
# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, question icon will be loaded.
# External url should start with http:// or https://
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

## 5.1 添加分类模块

1. 新建一个分类页面

```bash
$ hexo new page categories
```

2. 将 `source/categories/index.md` 文件中的 `type` 修改为 `type: "categories"`
3.   在菜单设置中将 `categories`取消注释
4.  打开 `scaffolds/post.md` 文件，在后面增加 `categories:`
5. 之后的每一篇文章会自动创建 `categories:` ,后面输入分类名即可

## 5.2 添加标签模块

1. 新建一个分类页面

```bash
$ hexo new page tags
```

2. 将 `source/tags/index.md` 文件中的 `type` 修改为 `type: "tags"`
3.   在菜单设置中将 `tags`取消注释
4.  打开 `scaffolds/post.md` 文件，在后面增加 `tags:`
5. 之后的每一篇文章会自动创建 `tags:` ，后面输入标签名即可，多个标签按如下格式输入

```bash
tags:
- 标签1
- 标签2
...
```

## 5.3 添加关于模块

1. 新建一个分类页面

```bash
$ hexo new page about
```

2. 修改 `source/about/index.md` 文件的内容为关于的内容即可
3.   在菜单设置中将 `about`取消注释

## 5.4 添加搜索模块

1. 安装 `hexo-generator-searchdb` 插件

```bash
$ npm install hexo-generator-searchdb --save
```

2. 打开 `站点配置文件` ，在最后添加

```bash
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

3. 打开 `主题配置文件`, 找到 `local_search` ,将 `enable` 修改为 `true`

```bash
# Local search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
```

## 5.5 修改个人社交信息

在 `主题配置文件` 中搜索 `social` ，选择想展示的社交信息，如下

```bash
social:
  GitHub: https://github.com/ShangguanHong || github
  E-Mail: mailto:sgh1450280694@gmail.com || envelope
  Weibo: https://weibo.com/5590338381 || weibo
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
```

# 6 网站效果

## 6.1 网站动画效果

1. 使用canvas_nest

在 `theme/next`目录下执行 `git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest` 命令，将 `主题配置文件` 中的`canvas_nest: false` 改为 `canvas_nest: true`

2. 使用three_waves

在 `theme/next`目录下执行 `git clone https://github.com/theme-next/theme-next-three source/lib/three_waves` 命令，将 `主题配置文件` 中的`three_waves: false` 改为 `three_waves: true`

3. 使用canvas_lines

将 `主题配置文件` 中的`canvas_lines: false` 改为 `canvas_lines: true`

4. 使用canvas_sphere

将 `主题配置文件` 中的`canvas_sphere: false` 改为 `canvas_sphere: true`

## 6.2 网站评论系统

请先登录或注册 [LeanCloud](https://leancloud.cn/), 进入控制台后点击左下角创建应用，进入刚刚创建的应用，选择左下角的`设置`>`应用Key`，然后就能看到你的`APP ID`和`APP Key`了。在 `主题配置文件` 中搜索Valine填写APP ID 和 APP Key即可

## 6.3 文章字数统计与阅读时间

1. 在网站根目录下运行命令

```bash
$ npm install hexo-symbols-count-time --save
```

2. 修改 `站点配置文件` ，添加以下代码

```bash
symbols_count_time:
symbols: true
time: true
total_symbols: true
total_time: true
```

2. 修改 `主题配置文件` ，找到 `symbols_count_time`

```bash
# Post wordcount display settings
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
symbols_count_time:
  separated_meta: true
  item_text_post: true
  item_text_total: false
  awl: 4
  wpm: 275
```