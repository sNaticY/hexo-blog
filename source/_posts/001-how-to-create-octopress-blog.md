---
layout: post
title: "Octopress(1)-在github上搭建octopress博客"
date: 2014-08-09 12:34
categories: Octopress折腾之路
tags: [octopress, github, Ubuntu]
---

马上就要毕业了。作为一个准挨踢攻城狮还是很有必要维护一个学习blog的。但是又不想使用各种广告满天飞的博客。所以自己搭一个才是高大上的选择。那么如何利用[github](http://github.com)搭建一个octopress博客呢。

<!--more-->

## PART 1 概述

接下来将详细讲述在ubuntu 14.04版本下安装octopress博客并生成静态页面并发布到GitHub Pages。如果大家刚开始使用ubuntu的话可能有以下工作要做。

- 安装ruby和git
- 配置github帐号
- 安装javascript运行环境
- 准备octopress
- 发布到github主页

## PART 2 开始

- ###安装git和ruby

``` bash
sudo apt-get install git ruby ruby-dev
```

修改gem的更新源。ruby官方源连接速度太慢。

``` bash
gem sources -a http://ruby.taobao.org/
gem sources -r http://rubygems.org/
```

- ###安装javascript运行环境

``` bash
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

## PART 3 配置

- ###配置github帐号

登录[github](http://github.com)，注册。帐号名假设为yourname。

在终端输入

```bash
ssh-keygen -t rsa -C "YOUREMAIL@EMAIL.com"
```

换成自己注册github时使用的email然后按照提示确认。在生成密钥之后打开 ～/.ssh/id_rsa.pub文件并将里边全部内容复制。在github右上角打开account setting页面。左侧选择SSH keys然后点Add SSH keys。将之前复制的内容粘贴到key，点击Add key。Title部分不填。

- ###配置octopress

下载octopress

```bash
git clone https://github.com/imathis/octopress.git octopress
```

下载完成以后开始配置

```bash
cd octopress
gem install bundler
bundle install
```

本地的octopress配置好以后进入github。右上角新建Repositories、命名为"yourname.github.io".

回到终端、进入octopress目录

```bash
rake setup_github_pages
```

按照提示输入刚才建好的Repo的SSH。

```bash
git@github.com:yourname/yourname.github.io.git
```

## PART 4 发布

- ###生成并发布octopress到git

在生成之前先配置一些基本信息。修改octopress根目录下的主配置文件_config.yml

``` yaml
url: http://yourname.github.io #你的博客地址
title:   #你的博客的标题
subtitle: #博客副标题
author: #作者
description:  #博客的简述
```

可以自定义的地方远不止这些、更多更高端的自定义的需求可以自行google相关信息。

自定义完成以后可以开始生成静态页面了。

```bash
rake new_post["post title"]  #生成一篇名为post title的博文
```

可在octopress/source/_posts文件夹中找到后缀为md的文件、在该文件末尾处填写博文正文

```bash
rake install                    #用于生成默认主题的页面模板
rake preview      #在浏览器中打开localhost:4000即可预览页面
rake generate                        #用于生成静态页面文件
```

预览页面确认无误以后即可发布到github上了

```bash
rake deploy
```

使用浏览器打开 http://yourname.github.com 尝试浏览。

- ###顺便利用github管理博客源文件

```bash
git add .  #注意add后面的空格和点
git commit -m "some changes"
git push origin source
```

## PART 5 总结

至此为止就成功的在本机上搭建octopress博客了。

- 使用`rake new_post["Post Title"]`生成一篇博客
- 编辑生成的md文件
- 使用`rake generate`生成静态页面
- 使用`rake deploy`发布。

可以看到整个过程总的来说还是很方便的。关于octopress的扩展将在以后的博客中进行更详细的介绍。

---

原文链接：https://snatix.com/2014/08/09/001-how-to-create-octopress-blog/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明
