---
title: 通过github分支备份Hexo源文件
date: 2019-06-26 13:08:15
copyright: true
categories: [Hexo学习笔记]
tags: [hexo]
---

# 1. 前言

利用hexo搭建博后（还不懂的朋友可以看这篇文章 [利用GithubPages + Hexo搭建自己的博客](https://shangguanhong.github.io/2019/05/28/GithubPages + Hexo搭建自己的博客/) ），仓库里只有生成的静态网页文件，是没有Hexo的源文件的。如果一不小心删除或者损坏了hexo源文件，那可就不得了了。这个时候就需要将hexo源文件也进行一下备份，我们这里就利用github的分支功能进行备份。

# 2. 配置

1. 在你的博客仓库创建一个分支hexo

![1561526447447](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1561526447447.png)

2. 切换默认分支为hexo

![1561526500792](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1561526500792.png)

3. clone博客仓库到本地

![1561526823075](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/1561526823075.png)

显示hexo表示使用的是hexo分支

3. 将之前的博客源文件中的 `_config.yml`  `themes` ,  `source` , `scffolds` , `package.json` , `.gitignore` 文件复制到博客仓库内

4. 将 `themes/next/` 中的`.git/`删除

5. 在博客仓库中执行 `npm install` ， `npm install hexo-deployer-git` 

6. 执行 `git add` ， `git commit -m "message"` ，`git push origin hexo` 来提交Hexo网站源文件

7. 执行hexo g -d 生成静态网页部署到github上。

   这样就可以使远程仓库的master分支保存静态网页，hexo分支保存源文件。

# 3. 修改

每次在本地对博客修改后，都要进行接下来两步

1、执行 `git add` ， `git commit -m "message"` ， `git push origin hexo` 来提交Hexo网站源文件

2、执行hexo g -d 生成静态网页部署到github上

# 4. 恢复

有的时候换电脑了，可以这样来恢复Hexo网站源文件
1、安装 `git`
2、安装 `Nodejs` 和 `npm`
3、clone博客仓库到本地
4、在博客仓库内执行命令 `npm install hexo-cli -g` 、 `npm install` 、 `npm install hexo-deployer-git` 

# 5. 参考资料

1. [怎么去备份你的Hexo博客](https://www.jianshu.com/p/baab04284923)