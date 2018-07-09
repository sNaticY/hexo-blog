---
title: Catlike学习笔记(1.1)-使用Unity实现一个钟表
draft: false
date: 2018-06-06 18:31:50
tags: [unity, tutorial, basic, csharp]
categories: Catlike学习笔记
---

最近发现『[Catlike系列教程](https://catlikecoding.com/unity/tutorials/)』觉得内容真的很赞，感觉有很多地方涉及到了我的知识盲点，如果真的可以照着做下来一遍的话应该收获颇丰。因为教程很长所以逐字翻译不太可能了（主要是翻译的太差）。基本上就是把实现的思路记录下来最后甩一个 「[Github Repo](https://github.com/sNaticY/CatlikePractice)」这样就可以了。理论上来说第一篇比较简单，感兴趣的同学可以移步「[原文链接](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/)」

<!--more-->

## PART 1 概述

实现一个钟表的话我们的目标就是

* 用一个拍扁的圆柱体制作表盘，用立方体制作刻度和时针分针，用一个细长的圆柱体制作秒针
* 写点 c# 脚本使其显示为当前时间
* 加点动画让指针平滑运动

## PART 2 制作场景

大家都是 Unity 熟手了所以具体制作流程就不讲了，博主自己也没仔细看就按照自己想法做了一个差不多的，具体思路就是多设一个层级然后父节点只旋转就可以把指针转到相应的位置而不需要同时调整 Rotation 和 Position。如果不是很懂的话可以回到「[原文地址](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/)」里面有更详细的做法，或者到我的「[Github Repo](https://github.com/sNaticY/CatlikePractice)」下载下来看看。

![MakeScene](http://ojgpkbakj.bkt.clouddn.com/2018060801.png)

## PART 3 写脚本控制指针

首先建立一个新的 MonoBehaviour 脚本比如说 ClockController.cs 之类的，把时针分针秒针的 Transform 的引用拖到脚本里，然后开始设置各个指针的位置。

那么众所周知表盘的360度被分割成12块所以每一块也就是每个小时占据了 360 / 12 = 30 度。同理每分钟占据了 360 / 60 = 6 度，每秒钟也是。那么直观来说就是这样写。

``` csharp
void Update()
{
	_hourArm.localEulerAngles = new Vector3(0, DateTime.Now.Hour * 30, 0);
	_minuteArm.localEulerAngles = new Vector3(0, DateTime.Now.Minute * 6, 0);
	_secondArm.localEulerAngles = new Vector3(0, DateTime.Now.Second * 6, 0);
}
```

然而运行一下会发现时针和分针都是笔直的指向其所在的时间。。如下图所示

![Clock](http://ojgpkbakj.bkt.clouddn.com/2018060802.png)

好吧现在刚好七点钟貌似看不出来，总之就是需要在比如 6:30 的时候时针应该指向 6 和 7 之间。所以这个度数需要再加上一点偏移，变成下面这样。

```csharp
void Update()
{
	var hour = DateTime.Now.Hour;
	var minute = DateTime.Now.Minute;
	var second = DateTime.Now.Second;
	var milisecond = DateTime.Now.Millisecond;
	_hourArm.localEulerAngles = new Vector3(0, hour * 30 + minute  / 60f * 30f, 0);
	_minuteArm.localEulerAngles = new Vector3(0, minute * 6 + second / 60f * 6f, 0);
	_secondArm.localEulerAngles = new Vector3(0, second * 6 + milisecond / 1000f * 6f, 0);
}
```

就可以轻松实现文章里的各种平滑移动之类的～

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018060803.gif)

## PART 4 总结

没有完全按照「[原文](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/)」中的写法来写好像这样会更简洁一点，然后大家应该也会更容易理解，不过作者的主要用意可能是想展示 coroutine 之类的吧不管那些了～总之大家可以进入「[Github Repo](https://github.com/sNaticY/CatlikePractice)」查看全部代码和运行 Demo。

---

原文链接：https://snatix.com/2018/06/06/019-gameobject-and-scripts//

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明