---
title: 重启Hexo(1)-在Mac上安装Hexo
date: 2017-01-08 17:40:20
tags: [hexo, mac, coding]
categories: Hexo博客重启计划
---

上次做过大量自定义的 [Hexo](http://hexo.io/) 博客，因为自己智障的原因全都搞丢了，伤心之余很久没有更新博客了，前段时间刚买了2016款 macbook pro ，想要在新电脑上搭建 hexo 环境重新开始写博客了～顺便把这个过程记录下来纪念博客重生历程好了！

<!--more-->

## PART 1 概述

其实在 mac os 上搭建 hexo 的过程是很简单的，遇到最大的阻碍是下载速度太慢～如果是完全干净没有装过 node 或者 npm 的机器的话跟着一步一步做就可以了，简单来说大概有以下几步。

1. 下载 Xcode 并安装 Command Line Tools
2. 安装 nvm 并使用 nvm 安装 npm + node
3. 安装 hexo
4. 简单配置一下
5. 在 coding.net 部署

## PART 2 环境准备

### 安装 Git

Mac用户直接使用以下 [Installer](https://sourceforge.net/projects/git-osx-installer/) 安装即可

### 下载 Xcode 并安装 Command Line Tools

根据 Hexo 官网对 Mac 用户的说明，因为 Hexo 的编译需要依赖 Xcode 的某些组件，所以需要先从 [App Store](https://itunes.apple.com/us/app/xcode/id497799835?ls=1&mt=12) 下载。下载完成后打开 `Xcode -> Preferences -> Download -> Command Line Tools -> Install` 即可。

### 安装 nvm 并使用 nvm 安装 npm + node

Hexo 官方推荐使用 [nvm](https://github.com/creationix/nvm) 来安装 npm，个人认为这样很有必要，因为 npm 版本太多太复杂，不是专业做 nodejs 开发很容易在日后使用其他工具的时候把环境搞乱或者遇到其他什么问题。这个过程非常简单～两种很方便的方式安装 nvm

cURL：

```bash
$ curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
```

Wget：

``` bash
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
```

等待 nvm 安装结束后，重启 Terminal 并执行以下命令来安装 Node.js

```bash
$ nvm install stable
```

## PART 3 Hexo 的安装与配置

### 安装 Hexo

完成基本的环境配置以后就可以开始安装 Hexo 了～很简单就一句话。

```bash
$ npm install -g hexo-cli
```

安装好以后在自己喜欢的地方初始化就好

```bash
$ hexo init blog
$ cd blog
$ npm install
```

到此为止 Hexo 的安装就全部完成了～ 运行`hexo server`，用浏览器打开 [http://localhost:4000/](http://localhost:4000/) 看看效果吧，基本的作者标题之类的配置就不再啰嗦啦大家自己看着办吧～

### 安装喜欢的主题

既然是个人博客那么定制主题就是一个很重要的环节～值得一提的是，为了避免像博主一样犯一些智障的错误导致自己辛苦定制的主题付之一炬的话，最好先找到自己喜欢的主题 fork，再安装～此后任何的修改都可以提交上去，下次想要在另一台机器上重新安装或者恢复的时候就很方便了。

在 [这个页面](https://hexo.io/themes/) 可以找到官方推荐的海量主题～从中挑选一款自己喜欢的吧，以博主目前使用的 [Material](https://github.com/viosey/hexo-theme-material) 为例。首先 Fork 一份到自己的仓库，如 [https://github.com/sNaticY/hexo-theme-material](https://github.com/sNaticY/hexo-theme-material) ，然后在 `blog` 目录下运行以下命令

```bash
$ git clone https://github.com/sNaticY/hexo-theme-material.git themes/material
```

之后打开`_config.yml`文件，修改其中的 `theme: landscape` 为

```yml
theme: material
```

到此为止新的主题就安装完成了～其他的更复杂的主题可能有一些插件需要安装，建议大家自行查看对应的 ReadMe 完成安装～

> P s. 通常主题作者都会持续维护自己开发的主题～修复一些 bug 或者提供一些新特性之类的，那么 fork 主题以后如何持续跟着作者的节奏更新呢？答案是创建一个 pull request 然后左边的 base fork 选择自己的仓库，右边 head fork 选择作者的仓库，然后再自行 Merge pull request 即可～

## PART 4 发布到 Coding.net

由于某种大家都懂的原因，国内访问 github-pages 服务非常慢。。因此作为替代方案，发布到 [coding.net](https://coding.net/) 是一个不错的选择。

### 创建仓库并添加 SSH公钥

登陆 coding.net，并创建一个与你的 id 同名的仓库，例如 [https://git.coding.net/sNatic/snatic.git](https://git.coding.net/sNatic/snatic.git) ，为你的账户配置 ssh。mac 用户直接在 Terminal 中输入

```bash
$ ssh-keygen -t rsa -C “username@example.com”
```

之后一路回车即可(喜欢的话也可以输入密码)，成功后会显示

```bash
Your identification has been saved in /Users/you/.ssh/id_rsa.
# Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

之后打开`id_rsa.pub`文件，复制其中全部内容，添加到 [账户->SSH公钥->新增公钥](https://coding.net/user/account/setting/keys) 的公钥内容中，点添加即可。

尝试验证一下有没有添加成功，输入`ssh -T git@git.coding.net`

```bash
$ ssh -T git@git.coding.net
The authenticity of host 'git.coding.net (45.124.125.220)' can't be established.
RSA key fingerprint is SHA256:jok1237q5LsfevE7iPNehBgXRw51ErE77S0av+Vg/Ik.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git.coding.net,45.124.125.220' (RSA) to the list of known hosts.
Hello sNatic! You've connected to Coding.net via SSH successfully!
```

出现类似以上文字就表示成功

### 配置 Hexo

首先 coding 是使用 git 进行托管的，因此我们需要下载 `hexo-deployer-git`工具

```bash
$ npm install hexo-deployer-git --save
```

接下来就可以在`_config.yml`中设置

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://git.coding.net/sNatic/snatic.git
  branch: master

```

回到 Terminal 中试试以下命令

```bash
$ hexo g && hexo d
```

如果没有任何报错的话，你的页面已经生成好并发布到 coding 了，最后一步就是回到 coding 中设置 coding-pages 服务。

### 设置 coding-pages 服务并自定义域名

进入之前创建好的 coding 项目的设置页面，选择部署来源为 master 分支，点击保存。等待部署完成以后可以根据提示进入 [http://sNatic.coding.me/sNatic](http://snatic.coding.me/sNatic) 访问你的博客了，理论上来说应该是和运行`$ hexo s`后访问 [http://localhost:4000/](http://localhost:4000/) 看到的页面完全一样的～

接下来进入你的域名 dns 服务商，以博主使用的 dnspod 为例。假设需要自定义域名为 `snatic.net` ，则添加一条 CNAME 记录，并将记录值设置为 `pages.coding.me`。如下图所示

![设置cname记录](http://ojgpkbakj.bkt.clouddn.com/2017010802.png)

回到 coding 项目的设置页面，在自定义域名中输入你的域名，点击绑定即可～

几分钟后 dns 生效以后就可以通过自定义域名访问你的博客了。

## PART 4 总结

到此为止一个全新的带有主题的 hexo 博客已经全部搭建完毕了，最后记得将工程源码托管到 github 哦～

---
原文链接：https://snatix.com/2017/01/08/007-install-hexo-on-mac/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明