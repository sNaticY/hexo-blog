---
layout: post
title: "Octopress(2)-自定义你的octopress博客"
date: 2014-08-12 15:40
categories: Octopress折腾之路
tags: [octopress]
---

在上一篇博客『[在github上搭建octopress博客](http://snatix.com/2014/08/09/001-how-to-create-octopress-blog/)』中讲到了如何在ubuntu环境下搭建octopress博客系统并装载到github上。那么相信大家都和我一样并不能满足于默认主题默认模板的博客。那么如何对我们的octopress博客进行定制呢？

<!--more-->

## PART 1 概述

想要让你的octopress博客与众不同大概有以下几种方法，首先最简单最直接的就是：

- 使用[第三方主题](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)

当然第三方主题也就那么几十种总有要求比较严格的朋友希望更个性化一些的选择：

- 在别人的主题基础上进行修改

当然只修改主题是远远不能满足需求的。还希望有到访人数统计啊、分享到微博空间朋友圈、添加评论系统、日志分类、搜索引擎关键词等等更多更高端的功能

- 添加第三方服务

目前为止想到的就是这些。其他需求什么的以后遇到了再考虑吧。

## PART 2 使用第三方主题

使用第三方主题的方法非常简单。我在github上找到这个页面『[3rd Party Octopress Themes](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)』里边收录了很多octopress主题。而且提供了preview。迅速从里边找到你心仪的主题然后安装吧。怎么安装？

- 点击你心仪的主题、进入到该主题的github页面。
- 在README.md里找到安装方法。

好吧有人看到这里要喷我了。这不是等于什么都没说么。好吧其实大部分主题可以通过以下方法来安装。以我写这篇博客时正在使用的octostrap3为例。

``` bash
cd octopress
git clone https://github.com/kAworu/octostrap3 .themes/octostrap3
rake 'install[octostrap3]'
rake preview   #预览一下确认无误以后再generate然后上传什么的
```

然后第三方主题就安装成功了，怎么样效果还可以吧。有一些主题默认添加了某些在我大天朝会被墙的服务什么twitter facebook什么的导致页面载入非常慢。所以推荐像我一样不会自己动手修改主题的尽可能选择简洁一些的。缺什么功能等我们以后学会了再补上就好。

## PART 3 修改主题样式

好吧其实到这我也什么都不会。所以机智的百度下发现这些

- 『[Octopress主题改造](http://shanewfx.github.io/blog/2012/08/13/improve-blog-theme/)』
- 『[为Octopress修改主题和自定义样式](http://www.360doc.com/content/12/0215/22/1016783_186940749.shtml)』

关于主题主题改造这方面这两位博主在文章里说的很清楚了。但是经过我的一系列尝试发现里边提到的关于`sass/custom/`这个文件夹下的文件取消注释进行修改的方式仅限于使用默认主题或者以默认主题为基础的第三方主题。很遗憾我已经安装了bootstrap系列的主题所以都没有用，就没办法为大家做具体的演示了。有需要的请移步上述博客。

## PART 4 添加第三方服务

嗯重要的部分终于来了。当初选择octopress大多是被简洁的风格和强大的可定制性吸引，于是我决定做以下几件事

- 在主页侧边栏添加 “about me”
- 在每一篇博客底部添加『[多说](http://duoshuo.com/)』评论模块
- 在所有页面侧边栏添加『Categories』模块
- 在所有页面侧边栏添加 "GitHub Repos"

> **P.S: 以下内容都是在octostrap3主题的基础上进行的。如果是其他主题可能页面效果并不好，请大家根据需要自行修改。**

### 主页侧边栏添加 “about me”

打开 `/source/_includes/custom/asides/about.html`
编辑内容如下

``` html
<section class="panel panel-default">
  <div class="panel-heading">
    <h3 class="panel-title">About Me</h3>
  </div>
  <div class="list-group">
  	<p class="list-group-item"><img src="/images/Me.png" width="230" height="230"></p>
    <!-- 在source/images/放置你的头像"Me.png",调整宽高看起来舒服即可 -->
  	<p class="list-group-item">你的自述1</p>
  	<p class="list-group-item">你的自述2</p>
    <!--自述3，4，5什么的随意添加-->
  </div>
</section>
```

考虑需要在哪些页面显示这些内容，我决定只在首页侧边栏的顶部显示，那么就打开`_config.yml`,找到`blog_index_asides:`去掉前面的“#”，添加

``` yaml
blog_index_asides: [custom/asides/about.html]
```

如果已有其他内容，注意用“,”和空格与后边的项目隔开。
保存后运行`rake preview`试试看效果吧。

### 添加『[多说](http://duoshuo.com/)』评论模块

首先在多说首页 http://duoshuo.com/ 注册帐号，记住你所添的多说站点的域名，如 yourname.duoshuo.com

在`source/_includes/post`目录下，新建文件 "duoshuo.html"

{% gist fcf0d3e2354bc663cb19 duoshuo.html %}

打开`source/_layouts/post.html`在`</article>`下插入如下代码。

{% gist fcf0d3e2354bc663cb19 post.html %}

最后`rake preview`看下有没有生效吧。

### 添加 “Categories” 模块

这个问题着实让我蛋疼了很久，百度一下可以找到很多，例如

- [为octopress添加分类(category)列表](http://codemacro.com/2012/07/18/add-category-list-to-octopress/)
- [帮你的octopress增加文章分类](http://blog.eddie.com.tw/2011/12/05/add-catetories-to-sidebar-in-octopress/)

但是这些并不符合要求，效果**非常**不美观。绝对无法容忍，然后就突然在octostrap主题的[官方页面](https://github.com/kAworu/octostrap3)
的[demo页](http://kaworu.github.io/octopress/)发现，作者的侧边栏的 categories 好漂亮。正是我想要的，果断找到了这个网页位于github的[源文件页面](https://github.com/kAworu/octopress/tree/octostrap3-demo)

找到`/source/_includes/custom/asides/category_list.html`
内容如下

{% gist fcf0d3e2354bc663cb19 category_list.html %}

然后在相同的目录下建立相同的文件全部copy进去。
然后打开`_config.yml`,找到`default_asides:`添加

``` yaml
default_asides: custom/asides/cate gory_list.html
```

嗯，这样效果就圆满了。

### 添加 GitHub Repos

这个就更简单了，直接打开`_config.yml`,修改如下：

``` yaml
# Github repositories
github_user: #你的github用户名
github_repo_count: #要显示的repo的数量
github_show_profile_link: true
github_skip_forks: true
```

在`default_asides:`添加

``` text
default_asides: asides/github.html
```

`rake preview`一下看看是不是圆满了。

## PART 5 总结

以上就是博主自己折腾的事情了。如果大家还有其他需要例如豆瓣、微博墙等等其他想要放进来的东西请参照以下几篇博客，应该可以找到满意的答案。

- [定制Octopress](http://blog.csdn.net/biaobiaoqi/article/details/9289563)
- [Octopress侧边栏定制及评论系统定制](http://812lcl.com/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)
- [在octopress中添加多说的最新评论](http://yrzhll.com/blog/2012/12/12/comment/)

---

原文链接：http://snatix.com/2014/08/12/002-customize-your-octopress-blog/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明