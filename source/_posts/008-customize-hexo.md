---
title: 重启Hexo(2)-Material深度探索
date: 2017-01-14 12:19:49
tags: [hexo, mac, blog]
categories: Hexo博客重启计划
thumbnail: http://ojgpkbakj.bkt.clouddn.com/2017011409.jpg
thumbnail_height: 489px
---

在上一篇文章「[在Mac上安装Hexo](http://snatix.com/2017/01/08/007-install-hexo-on-mac/)」中介绍了如何在 mac 搭建 Hexo 博客环境并部署到 coding.net 上，那么接下来的任务就是针对目前白纸一张的博客进行一些自定义让它变得更好用吧～我们的目标是基于 [Material](https://material.viosey.com/) 主题完成「标签云」「评论系统」以及「友情链接」并探索一些其他的小功能。

<!--more-->

## PART 1 概述

因为目前博主使用的是 [Material](https://github.com/viosey/hexo-theme-material) 主题，所以一切定制均以该主题为核心展开。其他主题可能步骤稍有不同但思路都差不多，大家酌情筛选就好~简单的来说主要有以下内容

1. 标签云页面
2. 接入多说评论系统
3. 添加友情链接页面
4. 添加搜索和生成二维码链接的功能
5. 为文章添加题图

## PART 2 标签云页面

Material 主题自带标签云功能，但是作者的教程页面藏的很深让我一度以为需要自行安装第三方标签云插件，具体使用方法非常简单，首先

``` bash
$ hexo new page "tags"
```

然后打开`source/tags/index.md`并添加标签

``` md
layout: tags
```

完成后保存文件，并在主题的`_config.yml`文件修改`pages`项，这样可以在左侧菜单中显示一个链接跳转至标签云页面

![标签云](http://ojgpkbakj.bkt.clouddn.com/2017011402.png)

```yaml
pages:
	标签云: "/tags"
```

这样标签云功能就轻松完成了，效果如图所示

![标签云](http://ojgpkbakj.bkt.clouddn.com/2017011401.png)

## PART 3 接入多说评论系统

Material 主题可以很方便的接入多说的评论，只需要修改主题`_config.yml`中的`comment`项即可：

```yaml
comment:
    use: duoshuo
    shortname: yourname
    duoshuo_thread_key: your_duoshuo_thread_key_fdas3be87
    duoshuo_embed_js_url: "https://yourname.duoshuo.com/embed.js"
```

那么`shortname`，`duoshuo_thread_key`，`duoshuo_embed_js_url`是如何获得的呢？首先进入 [多说官网](http://duoshuo.com/) 并使用任意方法登录。登录后点击「我要安装」按钮，填写必要的信息后点击创建即可～

创建站点完毕后进入站点，点击左侧「设置」-「基本设置」可看到站点的域名和密钥，域名前缀填写至`shortname`项，密钥填至`duoshuo_thread_key`项，并将`https://yourname.duoshuo.com/embed.js`替换为 `你的域名/embed.js`即可。

完成后评论系统效果如图所示

![标签云](http://ojgpkbakj.bkt.clouddn.com/2017011403.png)

## PART4 添加友情链接

友情链接同样也是 Material 主题自带的功能，作者好贴心～具体开启方法如下：

首先新建一个页面

``` bash
$ hexo new page "links"
```

然后打开`source/links/index.md`并添加标签

``` markdown
layout: links
```

在博客目录的`source`文件夹下新建`_data`目录，并新建文件`links.yml`，以以下内容为模板添加好友即可

``` yaml
Name: 
    link: http://example.com
    avatar: http://example.com/avatar.png
    descr: "这是一个描述"
```

最后编辑主题的`_config.yml`文件

``` yaml
pages:
	友情链接: "/links"
```

完成后的效果如下图所示，由于博主的好友较少，故借了主题作者的博客一张图：

![标签云](http://ojgpkbakj.bkt.clouddn.com/2017011404.png)

感觉超棒有没有～

## PART 5 探索其他有趣的功能

### 开启本地搜索服务

作者开启的默认搜索是 Google 搜索，但是由于大家都懂得原因其实并不是很好用，开启内置的本地搜索功能也非常方便，首先安装 [hexo-generator-search](https://github.com/PaicHyperionDev/hexo-generator-search) 插件

```bash
$ npm install hexo-generator-search --save
```

然后在主题的`_config.yml`文件中修改`search`项为

``` yaml
search:
    use: local
    swiftype_key:
```

这样就可以实现简单的搜索了，而且与主题融合的也很棒。

### 开启生成二维码服务

首先安装 [hexo-helper_qrcode](https://github.com/yscoder/hexo-helper-qrcode) 插件

```bash
$ npm install hexo-helper-qrcode --save
```

然后在主题配置文件`_config.yml`中修改

``` yaml
qrcode: true
```

效果如下图所示

![标签云](http://ojgpkbakj.bkt.clouddn.com/2017011405.png)

感觉是一个很有趣的小功能～

## PART 5 为文章添加题图

可能大家注意到，Material 这个主题每一篇文章都会默认随机配置一张主题图，显示在主页该文章的上方，如果想要自定义这张图怎么办呢

答案是在文章顶部的`front-matter`添加`thumbnail`即可，如

``` yaml
title: 重启Hexo(2)-Material深度探索
date: 2017-01-14 12:19:49
tags: [hexo, mac, blog]
categories: Hexo博客重启计划
thumbnail: http://ojgpkbakj.bkt.clouddn.com/2017011409.jpg
```

最终效果如本文题图所示。

## PART 6 总结

目前为止已经尽可能的挖掘 Material 这个主题中大多数有用的好东西了，可以愉快的开始写博客了(根本不想写搭完就满足的忘记了!!!)。修改好的配置记得上传到 github 进行托管哦，不然下次再来一遍感觉很浪费时间的。。这个篇章差不多也该完结了之后想到什么再补充吧~

---
原文链接：http://snatix.com/2017/01/14/008-customize-hexo/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明