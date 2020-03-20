---
title: "使用Travis CI自动部署hugo"
date: 2020-03-20T13:24:09+08:00
lastmod: 2020-03-20T13:24:09+08:00
draft: false
categories: [工具使用]
tags: ["Travis CI", "hugo"]
---

之前写过了一篇使用 `Travis CI` 自动部署hexo的文章([传送门]([https://shangguanhong.github.io/2019/12/19/github%E4%BD%BF%E7%94%A8travis-ci/](https://shangguanhong.github.io/2019/12/19/github使用travis-ci/)))，最近从hexo迁移到了hugo，那自然也是要使用一下自动部署工具来自动部署，操作基本和之前的一样，就是最后使用的`.travis.yml` 文件内容需要更改一下，因此如果不知道如何使用 `Travis CI`的可以查看之前的文章，这里就直接把 `.travis.yml` 的配置贴出来，修改部分参数即可直接使用。

**说明**：

- 使用的hugo源文件存放在hugo分支下
- 网站的静态文件存放在master分支下
- 

具体看如下配置

```yaml
language: go

go:
  - "1.14"  # 指定Golang 1.14

branches:
  only:
    - hugo  # 设置自动化部署的源码分支

# 设置环境变量
env:
 global:
  - version: 0.67.1
  - GH_REF: github.com/ShangguanHong/ShangguanHong.github.io/

before_install:
- export TZ='Asia/Shanghai'  # 设置时区

# 安装依赖组件
install:
  - wget https://github.com/gohugoio/hugo/releases/download/v${version}/hugo_${version}_Linux-64bit.tar.gz
  - tar -xzvf hugo_${version}_Linux-64bit.tar.gz
  - chmod +x hugo
  - export PATH=$PATH:$PWD
  - hugo version
  - git clone https://github.com/olOwOlo/hugo-theme-even.git themes/even
  - git log -p -2 | cat
  - commit_msg=$(git log -n1 --pretty=format:"%s")

script:
  - hugo

# after_script:
#   # 部署
#   - cd ./public
#   - git init
#   - git config user.name "ShangguanHong"
#   - git config user.email "sgh1450280694@gmail.com"
#   - git add .
#   - git commit -m "Update Blog By TravisCI With Build ${TRAVIS_BUILD_NUMBER}"
#   # Github Pages
#   - git push --force --quiet "https://$GH_TOKEN@${GH_REF}" master:master
#   # Github Pages
#   # - git push --quiet "https://$GH_TOKEN@${GH_REF}" master:master --tags

deploy:
  provider: pages # 重要，指定这是一份github pages的部署配置
  skip-cleanup: true # 重要，不能省略
  local-dir: public # 静态站点文件所在目录
  target-branch: master # 要将静态站点文件发布到哪个分支
  github-token: $GH_TOKEN # 重要，$GITHUB_TOKEN是变量，需要在GitHub上申请、再到配置到Travis
  # fqdn:  # 如果是自定义域名，此处要填
  keep-history: true # 是否保持target-branch分支的提交记录
  on:
    branch: hugo # 博客源码的分支
```

最后将 `.travis.yml` 文件一起上传到github的hugo分支上，即可做到自动部署网页到master分支。