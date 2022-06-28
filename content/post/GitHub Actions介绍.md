---
title: "GitHub Actions介绍"
date: 2022-06-25T12:58:34+08:00
lastmod: 2022-06-25T12:58:34+08:00
draft: false
categories: [工具使用]
tags: [GitHub]
---

## GitHub Acions 是什么

[GitHub Actions](https://github.com/features/actions) 是一个持续集成和持续交付 (CI/CD) 平台，可用于自动执行构建、测试和部署管道。GitHub 在 2018 年 10 月推出，在 2019 年 11 月后对该功能全面开放，现在所有的github用户可以直接使用该功能。

GitHub 提供 Linux、Windows 和 macOS 虚拟机来运行您的工作流程，或者您可以在自己的数据中心或云基础架构中托管自己的自托管运行器。在使用 Github Actions 之前首先需要了解以下前置知识:

- 持续集成/持续交付的概念
- Git 相关知识
- Linux 或 Windows 或 macOS 脚本相关知识
- Yaml 基础语法

大家知道，持续集成由很多操作组成，比如抓取代码、运行测试、登录远程服务器，发布到第三方服务等等。GitHub 把这些操作就称为 actions。

很多操作在不同项目里面是类似的，完全可以共享。GitHub 注意到了这一点，想出了一个很妙的点子，允许开发者把每个操作写成独立的脚本文件，存放到代码仓库，使得其他开发者可以引用。

如果你需要某个 action，不必自己写复杂的脚本，直接引用他人写好的 action 即可，整个持续集成过程，就变成了一个 actions 的组合。这就是 GitHub Actions 最特别的地方。

GitHub 做了一个[官方市场](https://github.com/marketplace?type=actions)，可以搜索到他人提交的 actions。另外，还有一个 [awesome actions](https://github.com/sdras/awesome-actions) 的仓库，也可以找到不少 action。

![img](https://www.wangbase.com/blogimg/asset/201909/bg2019091105.jpg)

上面说了，每个 action 就是一个独立脚本，因此可以做成代码仓库，使用`userName/repoName`的语法引用 action。比如，`actions/setup-node`就表示`github.com/actions/setup-node`这个[仓库](https://github.com/actions/setup-node)，它代表一个 action，作用是安装 Node.js。事实上，GitHub 官方的 actions 都放在 [github.com/actions](https://github.com/actions) 里面。

既然 actions 是代码仓库，当然就有版本的概念，用户可以引用某个具体版本的 action。下面都是合法的 action 引用，用的就是 Git 的指针概念，详见[官方文档](https://help.github.com/en/articles/about-actions#versioning-your-action)。

> ```bash
> actions/setup-node@74bc508 # 指向一个 commit
> actions/setup-node@v1.0    # 指向一个标签
> actions/setup-node@master  # 指向一个分支
> ```

## Github Actions 的基本概念

GitHub Actions 有一些自己的术语。

（1）**workflow** （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）**job** （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）**step**（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）**action** （动作）：每个 step 可以依次执行一个或多个命令（action）。

# workflow 文件介绍

GitHub Actions 的配置文件叫做 workflow 文件，存放在代码仓库的`.github/workflows`目录。

workflow 文件采用 [YAML 格式](https://www.ruanyifeng.com/blog/2016/07/yaml.html)，文件名可以任意取，但是后缀名统一为`.yml`，比如`foo.yml`。一个库可以有多个 workflow 文件。GitHub 只要发现`.github/workflows`目录里面有`.yml`文件，就会自动运行该文件。

workflow 文件的配置字段非常多，详见[官方文档](https://help.github.com/en/articles/workflow-syntax-for-github-actions)。下面是一些基本字段。

**（1）`name`**

`name`字段是 workflow 的名称。如果省略该字段，默认为当前 workflow 的文件名。

> ```
> name: GitHub Actions Demo
> ```

**（2）`on`**

`on`字段指定触发 workflow 的条件，通常是某些事件。

> ```
> on: push
> ```

上面代码指定，`push`事件触发 workflow。

`on`字段也可以是事件的数组。

> ```
> on: [push, pull_request]
> ```

上面代码指定，`push`事件或`pull_request`事件都可以触发 workflow。

完整的事件列表，请查看[官方文档](https://help.github.com/en/articles/events-that-trigger-workflows)。除了代码库事件，GitHub Actions 也支持外部事件触发，或者定时运行。

**（3）`on.<push|pull_request>.<tags|branches>`**

指定触发事件时，可以限定分支或标签。

> ```
> on:
>   push:
>     branches:    
>       - master
> ```

上面代码指定，只有`master`分支发生`push`事件时，才会触发 workflow。

**（4）`jobs.<job_id>.name`**

workflow 文件的主体是`jobs`字段，表示要执行的一项或多项任务。

`jobs`字段里面，需要写出每一项任务的`job_id`，具体名称自定义。`job_id`里面的`name`字段是任务的说明。

> ```yamL
> jobs:
>   my_first_job:
>     name: My first job
>   my_second_job:
>     name: My second job
> ```

上面代码的`jobs`字段包含两项任务，`job_id`分别是`my_first_job`和`my_second_job`。

**（5）`jobs.<job_id>.needs`**

`needs`字段指定当前任务的依赖关系，即运行顺序。

> ```YAML
> jobs:
>   job1:
>   job2:
>     needs: job1
>   job3:
>     needs: [job1, job2]
> ```

上面代码中，`job1`必须先于`job2`完成，而`job3`等待`job1`和`job2`的完成才能运行。因此，这个 workflow 的运行顺序依次为：`job1`、`job2`、`job3`。

**（6）`jobs.<job_id>.runs-on`**

`runs-on`字段指定运行所需要的虚拟机环境。它是必填字段。目前可用的虚拟机如下。

> - `ubuntu-latest`、`ubuntu-22.04`、`ubuntu-20.04`、`ubuntu-18.04`
> - `windows-latest`、`windows-2022`、`windows-2019`
> - `macos-latest`、 `macos-12`、`macos-11`、`macos-10.15`

下面代码指定虚拟机环境为`ubuntu-18.04`。

> ```
> runs-on: ubuntu-18.04
> ```

**（7）`jobs.<job_id>.steps`**

`steps`字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

> - `jobs.<job_id>.steps.name`：步骤名称。
> - `jobs.<job_id>.steps.run`：该步骤运行的命令或者 action。
> - `jobs.<job_id>.steps.env`：该步骤所需的环境变量。

下面是一个完整的 workflow 文件的范例。

> ```YAML
> name: Greeting from Mona
> on: push
> 
> jobs:
>   my-job:
>     name: My Job
>     runs-on: ubuntu-latest
>     steps:
>     - name: Print a greeting
>       env:
>         MY_VAR: Hi there! My name is
>         FIRST_NAME: Mona
>         MIDDLE_NAME: The
>         LAST_NAME: Octocat
>       run: |
>         echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
> ```

上面代码中，`steps`字段只包括一个步骤。该步骤先注入四个环境变量，然后执行一条 Bash 命令。

## GitHub Actions 中使用密文

在持续集成的过程中，我们可能会使用到自己的敏感数据，这些数据不应该被开源并泄露。那么如何才能安全的使用这些敏感数据呢? Github Actions 提供了 Secrets 变量来实现这一效果。我们可以在 github repo 上依次点击 Settings -> Secrets-> Actions->New repository secret创建一个敏感数据例如：PERSONAL_TOKEN， 然后我们就可以在 Github Actions 脚本中使用这一变量了:

```yaml
            - name: Deploy Web
              uses: peaceiris/actions-gh-pages@v3
              with:
                  personal_token: ${{ secrets.PERSONAL_TOKEN }}
                  PUBLISH_BRANCH: master
                  PUBLISH_DIR: ./public
                  commit_message: ${{ github.event.head_commit.message }}
```

这里的 secrets 就是一种 context，描述 CI/CD 一个 workflow 中的上下文信息，使用 ${{ expression }} 语法表示。[更多 context 信息可以参考官方文档](https://docs.github.com/cn/actions/learn-github-actions/expressions)

## GitHub Actions 执行结果

对于 GitHub Actions 的执行流程，我们可以点击 repo 上的 Actions 就可以看到 Action 的状态和执行结果等信息:

![image-20220626002757237](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/img/image-20220626002757237.png)

