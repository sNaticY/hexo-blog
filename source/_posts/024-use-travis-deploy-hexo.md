---
title: 重启Hexo(4)-使用travis-ci部署到Github和Coding
date: 2018-07-28 01:59:48
tags: [hexo, travis, github, coding, 部署]
categories: Hexo博客重启计划
description: 使用travis-ci部署Hexo博客到Github pages, 使用travis-ci部署Hexo博客到Coding Pages, Hexo博客持续集成, 自动部署Hexo博客
---

这周末要看 PUBG 的 PGI 比赛了，刚好在柏林举办受人之托去领一个饰品之类的所以时间非常紧所以『[Catlike学习笔记](https://snatix.com/categories/Catlike%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)』系列就先停更一个礼拜～写一篇比较简单的 Hexo 相关教程。其实之前就想写了，因为 Hexo 一直要从同一台设备部署，如果刚好手边没有设备又想改点什么东西又不想重新装一遍环境就很烦。。所以就想到用免费的 [travis-ci](https://travis-ci.org/sNaticY/hexo-blog/builds/408490063) 来帮助我们进行持续集成。这样甚至可以随时随地打开网页版 Github 修改文章非常方便～甚至还可以自动部署到 [coding.net](https://coding.net/) 上从而大幅度提高国内用户的访问速度。

<!--more-->

## PART 1 概述

作为「[Hexo博客重启计划](https://snatix.com/categories/Hexo%E5%8D%9A%E5%AE%A2%E9%87%8D%E5%90%AF%E8%AE%A1%E5%88%92/)」的第四篇，我们就不详细讲解如何搭建 Hexo 或者选择主题了，默认大家本地都有环境而且都已经部署好全部东西了，如果还没有的话欢迎移步「[重启Hexo(1)-在Mac上安装Hexo](https://snatix.com/2017/01/08/007-install-hexo-on-mac/)」安装与部署教程，以及「[重启Hexo(2)-Material深度探索](https://snatix.com/2017/01/14/008-customize-hexo/)」主题相关教程。那么我们今天的任务有以下几个

* 关联 Travis-CI 与 Github，并使用 Travis-CI 进行 Build
* 添加 push 到 Github pages 完成自动部署
* 添加 push 到 coding.net 完成自动部署

## PART 2 关联 Travis-CI 与 Github

关于 Travis-CI 具体介绍我们就不多讲了，想要了解更多的同学自行查找资料～我们首先需要进入 Travis-CI 注册帐号，一般来说只需要选择「使用 Github 帐号登录」就可以了完成所有关联了～此时大概会显示如下页面

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072802.png)

选择你的 Hexo 源码托管的仓库开启并点击`Settings`，只需勾选`Build Pushed Branches`其他的取消就好，不然其他人的 Pull Request 也会导致博客更新会容易出问题。。。如图所示

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072803.png)

最后在你的仓库根目录中添加`.travis.yml`文件如下

```yaml
language: node_js

node_js: stable

branches:
  only:
  - master

before_install:
- git config --global user.name "[你的Github帐号名]"
- git config --global user.email "[你的Github邮箱]"
- npm install -g hexo-cli

install:
- npm i

script:
- hexo clean
- hexo generate
```

完成以后将代码 Push 到 Github 上会发现已经可以自动 Build 了，关联成功的话 Build 过程中大概会这样。

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072804.png)

最后 Build 成功的话会在下方的 Log 中显示，这样一来我们的 Github 就与 Travis-CI 的关联完成了～

```bash
The command "hexo generate" exited with 0.
Done. Your build exited with 0.
```

## PART 3 Push到Github Pages完成自动部署

目前为止我们的 Travis-CI 具备了编译 Hexo 博客的能力，通过执行`hexo generate`指令将一大堆东西生成到`./public`目录中。按照通常方法我们会使用 hexo deploy 指令发布到固定的 Github Pages 仓库中。但这里似乎容易出现一些麻烦的问题，所以博主在这里采取了直接初始化仓库并强制提交的方法，简单粗暴不用改一堆配置，只需在之前创建的`.travis.yml`最后添加如下代码

```yaml
after_success:
- cd ./public
- git init
- git add --all .
- git commit -m "Travis CI Auto Builder"
- git push --quiet --force https://$REPO_TOKEN@[托管Hexo的Github Pages的仓库地址] master:master
```

前面四行代码大家都很容易理解，那么最后一行代码的「托管Hexo的Github Pages的仓库地址」是什么呢，比如说博主的 Hexo 仓库地址是`https://github.com/sNaticY/blog.git`，那么就最后一行就是`- git push --quiet --force https://$REPO_TOKEN@github.com/sNaticY/blog.git master:master`。大家还可以根据自身仓库状况修改要 Push 的分支之类的。

那么在这个之前出现的`$REPO_TOKEN`又是什么呢？首先我们的 Travis-CI 虽然获得了授权可以将所有的 Public 仓库的代码 Pull 下来进行编译，但是并没有权限 Push 代码，我们又不方便把密码或者 Token 之类的关键信息明文的写在 Public 仓库中的文件里。所以此处需要设置 **Environment Variables**。

首先进入 Github 点击自己的头像依次进入`Settings / Developer Settings / Personal access tokens`，点击`Generate new token`并设置名字勾选 Repo 相关权限，如图所示

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072806.png)

生成后复制 Token 并打开之前的 Settings 页面找到下图所示的位置，在`Name`框中输入`REPO_TOKEN`并将 Token 粘贴到`Value`框中。然后点击 Add。

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072805.png)

那么我们之前在`.travis.yml`中添加的`$REPO_TOKEN`就表示在该命令执行时自动调用此处的 Token 以确保仓库权限安全。全部完成后将`.travis.yml`提交到 Github，等待一小会儿如果 Log 最后扔显示` Done. Your build exited with 0.` 表示一切功能运行正常。打开你的博客看看，是不是已经生效了～

## PART 4 同时Push到Coding.net

具体新建仓库，创建 Coding Pages 绑定自定义域名之类的就不说了，我们的目标是让 Travis-CI 获得推送的权限，这样就可以直接 push 到 Coding Pages 的仓库完成部署。那么如何让 Travis-CI 获得权限呢？首先打开 [Coding.net](https://coding.net/) 点击右上角头像依次进入`我的帐号 / 访问令牌`并点击`新建令牌`，勾选`project:depot`

![picture](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018072807.png)

生成后与之前相同的方法设置 **Environment Variables**，设置`Name`为`CODING_TOKEN`。最后再在`.travis.yml`最后添加一行：

```yaml
- git push --quiet --force https://[Coding用户名]:$CODING_TOKEN@[你的Coding Pages仓库] master:master
```

提交后就发现任何时候提交到 Github 上的改动都会自动经过 Travis-CI 编译生成好并 Push 到 Github 和 Coding 上，让你可以随时随地在 Github 网页上修改文章并重新发布。

## PART 5 总结

本来想讲设置 DNS 的但是觉得跟第一篇教程有很多相同的东西显得非常重复就不浪费大家的时间了～不会设置的同学可以移步「[重启Hexo(1)-在Mac上安装Hexo](https://snatix.com/2017/01/08/007-install-hexo-on-mac/)」寻找 DNS 设置的方法。只需在 DNS 中区分国内国外的 IP 然后分别设置到 Github 和 Coding 就好了很简单～希望可以对大家有所帮助，下个礼拜回国休息几天希望可以多写几篇博客，嗯就这样大家拜拜～

---

原文链接：https://snatix.com/2018/07/28/024-use-travis-deploy-hexo/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明