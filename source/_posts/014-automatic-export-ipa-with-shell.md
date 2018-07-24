---
title: Unity自动化(3)-使用Shell脚本导出ipa
date: 2017-04-04 19:34:25
tags: [unity, mac, ipa, shell]
categories: Unity通用框架工程

---

本系列暂时弃坑 ~~可能永远都填不上了~~

<!--more-->

 ~~清明节在家睡了一天又看了一整天《生活大爆炸》发现只剩最后一天了- -12点起床洗澡吃饭以后，意识到每次假期都有一种刚起床就只剩最后一天的最后一个晚上的感觉好想哭。。。上周去了 UWA 最后就被迫鸽了大家一个周末，看来坚持每周写一篇文章的计划还是任重而道远啊～好了言归正传，我们回到自动化打包的话题，「[上一期](http://snatix.com/2017/03/18/013-export-ipa-without-apple-id/)」中我们提到由于各种原因我们仅有 mobileprovision 文件和 p12 文件来导出 ipa的方法，感觉其实是很麻烦的，那么我们的目标就是让这一切自动化起来。~~

~~<!--more-->~~

## ~~PART 1 概述~~

~~我们的目标是，从 svn update 以后就可以直接运行一个 shell 脚本导出 ipa，无需任何额外操作。那么其实中间大概有这么几个步骤。~~

1. ~~打开 Unity 选择 Build XcodeProject~~
2. ~~打开 Xcode 做一大堆配置~~
3. ~~XcodeProject Archive~~
4. ~~导出 ipa~~

~~在写脚本的时候博主就尽量遵循简单易懂的原则方便大家阅读，尽量不写多余的控制逻辑，有需要的话大家可以自行扩展~首先在开始之前确保我们已经按照上一篇教程的方式成功导出过 ipa 从而确认环境没有太大的问题。~~

## ~~PART 2 Build XcodeProject~~







---

原文链接：https://snatix.com/2017/04/04/014-automatic-export-ipa-with-shell/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明