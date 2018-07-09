---
layout: post
title: "Octopress(3)-使用gitcafe提高访问速度"
date: 2014-08-13 10:14
categories: Octopress折腾之路
tags: [octopress, github, gitcafe]
---

在大家学习了上一篇博客『[自定义你的octopress博客](http://snatix.com/2014/08/12/002-customize-your-octopress-blog/)』，那么大家会发现其实使用 GitHub Pages 访问速度并不是那么理想。今天发现了一个酷似 GitHub 的国内站点叫 "[Gitcafe](http://gitcafe.com)"，访问速度自然比 github 要快得多。而且 gitcafe 也提供了 gitcafe pages 可用于搭建 octopress 博客。那么我们如何同时使用 gitcafe 和 github托管我们的博客呢？

<!--more-->

## PART 1 概述

首先我们假设我们的ubuntu已经安装git和octopress并且我们的博客已经在 GitHub pages 中成功运行了。那么我们如何在 gitcafe pages 中创建完全同步的页面用于国内访问提速呢。

- 注册 gitcafe 并创建 gitcafe pages 项目
- 添加 gitcafe 的 SSH 公钥
- 修改`Rakefile`使`rake deploy`命令可同时上传页面到 github 和 gitcafe。
- 修改`/.git/config`使 `git push origin source` 可同时源代码提交到 github 和 gitcafe
- 修改dns服务使我们使用国内网络连接到你的域名时默认访问 gitcafe pages 从而提高速度

最后一条大家慎用，可能会严重破坏使用习惯，那是博主为了偷懒自己改的。正确方法可以参照『[同步github上的项目到gitcafe](http:/blog.csdn.net/forever_wind/article/details/37506263/)』

## PART 2 注册 gitcafe 并创建项目

1. 打开 [gitcafe](http://gitcafe.com) 并点击 sign up
2. 填入你的 EMAIL USERNAME PASSWORD 什么的。下方有语言选择实在不会的点一下简体中文应该没问题了- -
3. 点左侧的『 + 创建 』这个按钮可以创建一个新项目（我也不知道如果没有项目的时候这个按钮在哪。可能在更显眼的位置吧总之应该都能找到）
4. 项目名必须和用户名(就是上面的『拥有者』这一项)中的内容完全一致，这点与 github 不同
5. 完成

感觉很简单应该不会出现问题

## PART 3 添加 gitcafe 的 SSH 公钥

有很多教程可以找到但都涉及到要删除原来的公钥。嗯我们可不是以后再也不用 github 了只是想添加 gitcafe 作为国内的镜像。所以最终我在 gitcafe 官网找到『[如何使用多个公密钥](https://gitcafe.com/GitCafe/Help/wiki/如何同时使用多个公秘钥)』。基本步骤如下

在终端输入以下命令。要把 YOUR_EMAIL@YOUREMAIL.COM 改成你自己在注册 gitcafe 时填写的邮箱

``` bash
ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM" -f ~/.ssh/gitcafe
```

生成过程中会出现以下信息，按屏幕提示操作设置路径和口令

``` bash
$ ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM" -f ~/.ssh/gitcafe
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/username/.ssh/gitcafe.
Your public key has been saved in /c/Users/username/.ssh/gitcafe.pub.
The key fingerprint is:
15:81:d2:7a:c6:6c:0f:ec:b0:b6:d4:18:b8:d1:41:48 YOUR_EMAIL@YOUREMAIL.COM
```

该过程将会在`~/.ssh/`目录下生成`gitcafe`和`gitcafe.pub`两个文件。然后打开`~/.ssh/config`并输入以下内容(没有就新建一个)

``` bash
Host gitcafe.com www.gitcafe.com
  IdentityFile ~/.ssh/gitcafe
```

然后打开`~/.ssh/gitcafe.pub`，复制全部内容。进入 gitcafe 右上角『账户设置』里的『SSH 公钥管理』，点击『添加新公钥』然后把之前复制的内容粘贴进去确定即可。

完成以后可以通过以下命令测试连接

``` bash
ssh -T git@gitcafe.com
```

按照提示输入`yes`然后输入刚才创建 SSH 时设置的口令。如果出现以下信息就表示连接成功。

``` bash
Hi USERNAME! You've successfully authenticated, but GitCafe does not provide shell access.
```

## PART 4 修改 Rakefile

修改 Rakefile 的目的在于我们只需要一条指令就是`rake deploy`即可将页面一次发布到 github 和 gitcafe 两个代码仓库，可以节省很多时间。当然有不怕麻烦的可以手动进入到`_deploy`文件夹然后 push 整个文件夹到 gitcafe 。

找到 octopress 根目录的`Rakefile`文件并用文本编辑器打开。有如下代码

``` ruby
cd "#{deploy_dir}" do
    system "git add -A"
    message = "Site updated at #{Time.now.utc}\n\n[ci skip]"
    puts "\n## Committing: #{message}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing generated #{deploy_dir} website"
    Bundler.with_clean_env { system "git push origin #{deploy_branch}" }
    puts "\n## Github Pages deploy complete"
    #添加下面这两行代码，YOURNAME记得改
    system "git remote add gitcafe git@gitcafe.com:YOURNAME/YOURNAME.git >> /dev/null 2>&1"
    system "git push -u gitcafe master:gitcafe-pages"
  end
```

这样以后就可以还像以前一样使用`rake deploy`了。

## PART 5 修改`/.git/config`

这一步是博主为了偷懒而自己改的。反正以后`push`到两边都可以通过一条指令来了。如果有需要单独 push 到 github 或者 gitcafe 的请按照『[同步github上的项目到gitcafe](http:/blog.csdn.net/forever_wind/article/details/37506263/)』的指导完成。

同样还是找到 octopress 根目录的 `.git/config`文件用文本编辑器打开(找不到的话按ctrl+H可以显示隐藏文件)

``` text
[remote "origin"]
  url = git@github.com:snatic0/snatic0.github.io.git
	#添加下面这一行代码，同样要改 YOURNAME
  url = git@gitcafe.com:YOURNAME/YOURNAME.git
  fetch = +refs/heads/*:refs/remotes/origin/*
```

完成以后就可以还像博主的[第一篇博文](http://snatix.com/2014/08/09/001-how-to-create-octopress-blog/)里介绍的那样通过下面三条指令把博客的 octopress 源代码 push 到两个代码仓库进行版本管理了。（貌似也没什么必要的样子）

``` bash
git add .  #注意add后面的空格和点
git commit -m "some changes"
git push origin source
```

## PART 6 修改DNS服务

修改DNS服务的目的在于我们可以通过以前的域名优先访问到我们位于 gitcafe pages 的博客从而提高访问速度。然后在国外则会默认访问 github pages 速度也很赞。可谓两全其美。

首先进入到 gitcafe 中你的项目首页。然后点右上的『项目管理』。进入以后在左侧点击『自定义域名』。最后添加你的域名。

![自定义域名](http://ojgpkbakj.bkt.clouddn.com/2014081303.png)

进入到你的域名的dns服务商的页面。博主使用的是『[Dnspod](http://www.dnspod.cn/)』将以前的 github pages 的 CNAME 记录的线路类型设为默认，然后新添加三条 A记录，线路类型分别是『电信』『联通』『教育网』，记录值都是`117.79.146.98`。等待一会儿就可以了。

![DNSpod](http://ojgpkbakj.bkt.clouddn.com/2014081301.png)

想知道是否成功可以ping一下看看

![PING](http://ojgpkbakj.bkt.clouddn.com/2014081302.png)

速度是不是快了不少呢~

---

原文链接：http://snatix.com/2014/08/13/003-use-gitcafe-to-speed-up/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明