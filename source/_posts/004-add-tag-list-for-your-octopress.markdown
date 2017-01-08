---
layout: post
title: "Octopress折腾之路(4)-为octopress博客添加更多功能"
date: 2014-08-15 19:39
categories: Octopress折腾之路
tags: [octopress]
---

在本系列的第二篇『[自定义你的octopress博客](http://snatic.tk/blog/2014/08/12/customize-your-octopress-blog/)』中我们为 octopress 做了一些自定义。但是博主发现如果没有标签系统来管理博文，仅仅依靠 categories 是远远不够的。所以这次我们来试试为 octopress 添加标签系统，顺便为博客做一些SEO让搜索引擎更容易的收录到我们的页面。最后优化以下rakefile以减少不必要的劳力。那么

<!--more-->

## PART 1 概述

这次要提到的内容比较少感觉已经不需要『概述』这个模块了，但是为了风格统一还是稍微写一些。那么我们大概要做以下几件事情。

- 侧边栏添加Tag list并为博文添加标签
- 优化 rakefile

## PART 2 添加标签系统

随便搜一下就可以看到很多关于标签系统的博文了。但是貌似大家提到的都是中文支持有问题的那个。找来找去找到[812lcl](https://github.com/812lcl)大神的这篇博文『[Octopress侧边栏及评论系统定制](http://812lcl.com/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)』。感谢这位大神自改了一份『[支持中文的tag系统](https://github.com/812lcl/category_tag)』放到了 github 上。大神说下载下来 clone 到 octopress 目录即可。但经博主亲测还需要一些工作要做。

注意备份我们之前改好的 `source/_includes/custom/category_list.html`，不然会覆盖掉。还有在
[原插件](https://github.com/robbyedwards/octopress-tag-pages)里提到貌似还有两个文件需要创建。

{% gist d50e4ce598cedb393c9d tag_index.html %}

在`source/_layout`中新建`tag_index.html`复制以上内容即可，然后在`source/_includes/custom/`中新建`tag_feed.xml`然后复制以下内容

{% gist d50e4ce598cedb393c9d tag_feed.xml %}

最后在`_config.yml`中找到`default_asides`添加

``` text
custom/asides/tag_list.html
#或者是你想添加的其他的比如tag_cloude.html
```

然后还要找到`_config.yml`

``` yaml
category_dir: blog/categories
#添加下面这一行
tag_dir: blog/tags
```

这样就可以preview一下看看效果了,怎么样效果有是有了不过跟我们的主题完全不符好吧需要好好改造一下。下面是博主自改的适合 octostrap3 这个主题的`tag_list.html`，需要标签云的也可以自己尝试改一下

{% gist d50e4ce598cedb393c9d tag_list.html %}

最后我们的octopress支持tag了我们该怎么为文章添加tag呢，只要在每篇博文的最前自动生成的`categories: `下添加`tags: [tag1, tag2, tag3]`就可以添加了

## PART 3 优化 Rakefile

我们可以在每篇博客中都为搜索引擎提供更多的信息来帮助搜索引擎更准确的捕捉到你的博客。方法呢就是在每篇博文的前面添加像这样的信息

``` yaml
layout: post
title: "Octopress折腾之路(4)-为octopress博客添加更多功能"
date: 2014-08-15 19:39
comments: true
categories: Octopress折腾之路
description: Octopress折腾之路(4)-为octopress博客添加更多功能
keywords: octopress, 添加， Tag, 标签. SEO, 自定义, 新特性
tags: [octopress]
```

但是手动添加实在是太麻烦了，我们可以尝试修改Rakefile从而让每篇博文自动添加这些东西，刚好可以顺便解决`rake new_post`命令中处理中文会出bug的问题

打开 Rakefile 在文件底部加入以下代码

``` ruby
desc "Begin a new post in #{source_dir}/#{posts_dir} with Alias"
task :post, :title, :title_alias do |t, args|
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  args.with_defaults(:title => 'new-post')
  title = args.title
  title_alias = args.title_alias
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title_alias}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "description: #{title_alias}"
    post.puts "keywords: "
    post.puts "tags: "
    post.puts "---"
  end
end
```

以后就可以使用`rake post['英文名','中文名']`这个命令来自动生成一个后缀为 md 的文件，打开以后很多信息已经写好了，稍微填一下就可以开始写博客了会比以前方便很多。

## PART 4 总结

终于，我们的『Octopress折腾之路』告一段落了。折腾到现在也差不多该稳定一段时间了。等新的问题出现博主在解决的时候继续更新本专题吧。按照『[Road of Octopress](http://snatic.tk/blog/categories/road-of-octopress/index.html)』专题中的内容一步一步的执行下来就可以搭建一个像『[安哥拉大胖兔的窝](http://snatic.tk)』这样的 blog 了，最后还有一些尚未解决的问题不知道有没有大神可以告知。

- 『[Octostrap3](https://github.com/kAworu/octostrap3)』 的作者自己写的适用于该主题的 categories 并不支持中文，虽然直接使用默认的 categories 可以支持。博主完全不懂 ruby 不懂 HTML 想稍微改一改也无从下手。
- 在代码块中的｛% 各种不同的代码 code %｝语句无论怎样处理都一定会执行，博主不得不使用上传代码到gist然后再引用的方式解决，导致某些文章加载速度变慢影响体验。
- 还没想到。。之后再补吧

---

原文链接：http://snatix.com/2014/08/15/004-add-tag-list-for-your-octopress/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明