---
title: 快节奏多人游戏同步(3)-Entity插值
date: 2018-05-18 14:40:00
tags: [多人游戏, 同步, 客户端, 服务器]
categories: 快节奏多人游戏同步
---

第三篇也新鲜出炉了～之前随便找了下发现其实这个系列已经有人翻译过了，翻译的也还可以但是坑已经开始挖了最好就顺便填完吧反正也没有很深。不管怎么说也是认认真真的重新翻译了一遍希望相比其他的翻译可以更上一层楼，虽然有些名词感觉不管怎么翻译都怪怪的干脆就保留原文说不定更容易理解一点。最后依然贴出「[原文地址](http://www.gabrielgambetta.com/entity-interpolation.html)」大家不放心或者不太理解的地方直接查看原文就好。

<!--more-->

## PART 1 概述

在本系列的「[第一篇文章](http://snatix.com/2018/05/07/015-client-server-game-architecture/)」中，我们介绍了关于权威服务器及其反作弊特性，然而仅仅是最简单的实现可能会导致关于可玩性和响应速度的问题。在「[第二篇文章](http://snatix.com/2018/05/14/016-client-side-prediction-and-server-reconciliation/)」中，我们提出了「客户端预测」的方案来克服这个困难。

以上两篇文章事实上介绍的是一种在连接到有传输延迟的远程权威服务器的状况下，可以让玩家像单机游戏一样流畅的控制角色移动的一种概念和技术。

在本文中，我们将会讨论在在同一台服务器上有「其他玩家控制的角色」的情况。

## PART 2 服务器 time step

在之前的文章中，我们的服务器的实现非常简单，它负责处理客户端的输入并更新游戏状态，最后将结果发送回客户端。但是当多个玩家连接的时候，服务器主循环会有略微不同。

在多个客户端连接到同一台服务器的情况下，每个客户端可能会以相当快的频率发送输入（取决于玩家发送指令的频率，比如按下按键，移动鼠标或者点击屏幕之类的）每次收到客户端输入就更新游戏状态并广播给全部玩家可能会消耗过多的 cpu 资源和带宽。

更好的方案是将客户端的输入放进队列中先不处理，等待服务器周期性的更新，例如每秒钟10次之类的。这100ms 我们便称之为服务器的 time step。每次更新都会将所有未被处理的输入进行处理，最后将新的游戏状态广播给所有玩家。

总而言之，整个游戏会以固定的，不受客户端输入的指令内容和数量影响的频率持续更新。

## PART 3 低频更新处理

从客户端的角度来看，这种方法像以前一样可用，因为客户端预测独立于延迟更新之外，所以很明显在相对低频的状态更新下仍然有效。然而由于游戏状态以较低频率广播（例如100ms），客户端只能得到较少的其他正在移动的实体的信息。

原本的实现将会在收到其他角色的位置信息时立即更新其状态，这将会导致角色每 100ms 顺移一次而非平滑移动。

![Client 1 as seen by Client 2.](https://s2.51cto.com/wyfs02/M02/8D/AE/wKiom1ilt5DD1uwkAACfpevkX3A342.png-wh_500x0-wm_3-wmp_4-s_4138866562.png)

根据你所开发的游戏类型的不同，这个问题有很多不同的解决方案，你的游戏中的实体可预测性越高，你就越容易得出正确的结果。

## PART 4 航位推测法

假设你正在制作一个赛车游戏，一辆跑得非常快的车是很容易预测的，比如说辆车的速度是 100m/s ，一秒后这辆车的位置大概就是其前方100m处。

为什么是『大概』呢？因为在这一秒钟之间这两车可能加速或者减速了一点点，或者左拐右拐了的一点点，这里的关键词是『一点点』。无论玩家做了什么操作，高速行驶的车辆的机动性很大程度上取决于它之前的位置，方向和速度。换而言之，一辆车不能瞬间180度大转弯～

为什么这种方法适用于每 100ms 发送一次消息的服务器呢？客户端收到每一个对手车辆的已被验证过的速度和方向后，接下来的100ms之内并不会收到其他任何消息，但是客户端仍然需要将移动的过程显示出来，最简单的方式就是假设车辆的速度和航向在这100ms内保持不变，然后在本地移动车辆的物理位置。然后在100ms后，服务器返回消息时再矫正车辆的位置。

这里的「矫正」可能很大也可能很小，取决于许多变量。如果玩家直直的开着车完全没有拐弯也没有踩刹车和油门，那么预测的位置将会与最终位置完全匹配。然而玩家撞到什么东西的话，预测出来的位置将会产生巨大的误差。

需要注意的是，航位推测法可以适用于低速情形，比如战列舰等。事实上「航位推测法」这个词本来就来自于海军导航术语。

## PART 5 实体插值

有一些情形「航标推测法」是无法适用的，尤其是那些玩家的方向和速度会突然变化的那种。。比如说在3d射击游戏中，玩家经常会很以极快的速度走走停停转方向，由于位置和速度不再是可预测的，使得航标推测法完全没用了。

在服务器下发数据的时候才更新位置是不行的，因为这样会导致玩家每100ms闪现一次，严重影响正常游戏～

目前的状况是客户端只知道每 100ms 的玩家的准确位置，因此关键在于如何在两个 100ms 之间把发生的事情「脑补」出来。解决这个问题只需要将其他玩家相对于玩家自己的“过去”的位置显示出来就好了。

假设你在 **t = 1000** 收到了所有玩家的位置信息，因为 **t = 900** 时的游戏状态也已经收到了，所以你同时知道某个玩家 **t = 900** 和 **t = 1000** 时的位置，所以从 **t = 1000** 到 **t = 1100** ，你可以显示其他玩家 **t = 900** 到 **t = 1000** 这段时间内做的事情。如此你便一直显示着你自己的确切位置，而其他玩家则晚 100ms 才更新。

![Client 2 renders Client 1 in the past, interpolating last known positions.](http://www.gabrielgambetta.com/img/fpm3-02.png)

用来从 **t = 900** 到 **t = 1000** 插值的位置信息取决于游戏本身，一般来说插值法表现还不错。如果不满意的话，可以让服务器发送更多的细节给你，比如说发送玩家在移动过程中的片段序列，或者每10ms进行一次采样从而让插值结果更加自然。（发送的数据量并不一定是原来的10倍，尝试改成发送增量式的位置可以在特定情况下可以得到极大的优化）

需要注意的是使用这种技术，每个玩家看到的游戏世界可能会有略微不同。因为每个玩家都看到自己当前状态但是看到的是其他玩家的过去状态。即使是快节奏游戏中也是如此。不过一般来说100ms的延迟不是非常明显。

有一种例外，当你需要很精确的计算的时候，比如说一个玩家开枪射击另一个玩家，因为你看到的是其他玩家是过去的位置，导致你的瞄准总是延迟了 100ms ，这就意味着你瞄准的总是目标100ms之前的位置，我们在下一篇文章中会处理这个问题。

## PART 6 总结

即便是低频更新和网络延迟的情况下，你也必须让玩家感受到持续的平滑移动。在「[第二篇文章](http://snatix.com/2018/05/14/016-client-side-prediction-and-server-reconciliation/)」中，我们讲解了客户端预测和服务器调和方法来让玩家实时移动以确保玩家的输入会得到立即反馈，从而避免的延迟造成的不适感。

但是其他的玩家控制的实体依然有问题，在本文中我们介绍了两种方法来解决。

第一种是 「航位推测法」 在特定种类的模拟游戏中实体的位置可以根据当前的速度和加速度等等进行预测。但是在条件不符合的时候就会导致预测失败。

第二种 「实体插值法」 完全不预测未来的位置，只使用实体的真实数据然后将其显示的时间略微延迟。

这种方法唯一的影响就是玩家同时看到了自己「*现在的*状态」和其他人「*过去的状态*」，这通常可以带来天衣无缝的体验。

然而仅仅是这样的话，当某个事件需要被极其精确的处理的时候就会出现问题，比如说当你射击移动物体时，「客户端2」屏幕上显示的「客户端1」的位置既不是服务器中「客户端1」的位置也不是「客户端1」自己显示的位置。所以爆头什么的变成了不可能的任务，然而类似设定在各种游戏中都必不可少，因此我们会在下一篇文章中解决这个问题。

------

原文链接：https://snatix.com/2018/05/18/017-entity-interpolation/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明