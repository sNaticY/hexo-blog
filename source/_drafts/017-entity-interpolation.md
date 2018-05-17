---
title: 快节奏多人游戏同步(3)-Entity插值
date: 2018-05-07 17:38:57
tags: [multiplayer, sync, client, server]
categories: 快节奏多人游戏同步
---

[<< Series Start](http://www.gabrielgambetta.com/client-server-game-architecture.html)[Gabriel Gambetta](http://www.gabrielgambetta.com/index.html)

# Fast-Paced Multiplayer (Part III): Entity Interpolation 实体插值

[Client-Server Game Architecture](http://www.gabrielgambetta.com/client-server-game-architecture.html) · [Client-Side Prediction and Server Reconciliation](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html) · [Entity Interpolation](http://www.gabrielgambetta.com/entity-interpolation.html) · [Lag Compensation](http://www.gabrielgambetta.com/lag-compensation.html) · [Live Demo](http://www.gabrielgambetta.com/client-side-prediction-live-demo.html)

## Introduction 概述

在本系列的第一篇文章中，我们介绍了关于权威服务器及其反作弊特性，然而仅仅是最简单的实现可能会导致关于可玩性和相应速度的问题。在第二篇文章中，我们提出了「客户端预测」的方案来克服这个困难。

In the [first article](http://www.gabrielgambetta.com/client-server-game-architecture.html) of the series, we introduced the concept of an *authoritative server* and its usefulness to prevent client cheats. However, using this technique naively can lead to potentially showstopper issues regarding playability and responsiveness. In the [second article](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html), we proposed *client-side prediction* as a way to overcome these limitations.

以上两篇文章介绍的其实是一种即使是连接到有传输延迟的远程权威服务器也可以让玩家像单机游戏一样流畅的控制游戏内角色移动的一种概念和技术。

The net result of these two articles is a set of concepts and techniques that allow a player to control an in-game character in a way that feels exactly like a single-player game, even when connected to an authoritative server through an internet connection with transmission delays.

在本文中，我们将会讨论在在同一台服务器上有其他玩家控制角色的情况。

In this article, we’ll explore the consequences of having other player-controled characters connected to the same server.

## Server time step 服务器时间步进

在之前的文章中，我们的服务器实现非常简单，它负责处理客户端的输入并更新游戏状态，最后将结果发送回客户端。但是当多个玩家连接的时候，服务器主循环会有略微不同。

In the previous article, the behavior of the server we described was pretty simple – it read client inputs, updated the game state, and sent it back to the client. When more than one client is connected, though, the main server loop is somewhat different.

在这种情况下，多个客户端可能会相当快节奏的同时发送输入（和玩家发送指令一样快，比如按下按键，移动鼠标或者点击屏幕之类的）每次收到客户端输入就更新游戏状态并广播给全部玩家可能会消耗过多的 cpu 资源和带宽。

In this scenario, several clients may be sending inputs simultaneously, and at a fast pace (as fast as the player can issue commands, be it pressing arrow keys, moving the mouse or clicking the screen). Updating the game world every time inputs are received from each client and then broadcasting the game state would consume too much CPU and bandwidth.

更好的方案是将客户端的输入放进队列中先不处理，等待周期性的更新，例如每秒钟10次之类的。这100ms的间隔我们称之为 time step 步进。每次更新都会将所有未被处理的输入进行处理，然后将新的游戏状态广播给所有玩家。

A better approach is to queue the client inputs as they are received, without any processing. Instead, the game world is updated periodically at low frequency, for example 10 times per second. The delay between every update, 100ms in this case, is called the*time step*. In every update loop iteration, all the unprocessed client input is applied (possibly in smaller time increments than the time step, to make physics more predictable), and the new game state is broadcast to the clients.

总而言之，整个游戏会以固定的，不受客户端的表现和数量影响的频率持续更新。

In summary, the game world updates independent of the presence and amount of client input, at a predictable rate.

## Dealing with low-frequency updates 低频更新处理

从客户端的角度来看，以上方法可以像以前使用客户端独立预测一样的平滑，所以很明显在相对低频的状态更新下仍然是可预测的。然而由于游戏状态以较低频率广播（例如100ms），客户端只能得到较少的其他移动实体的信息。

From the point of view of a client, this approach works as smoothly as before – client-side prediction works independently of the update delay, so it clearly also works under predictable, if relatively infrequent, state updates. However, since the game state is broadcast at a low frequency (continuing with the example, every 100ms), the client has very sparse information about the other entities that may be moving throughout the world.

原来的实现将会在收到其他角色的位置时立即更新，这将会导致角色每 100ms 顺移一次而不是平滑移动。

A first implementation would update the position of other characters when it receives a state update; this immediately leads to very choppy movement, that is, discrete jumps every 100ms instead of smooth movement.

![Client 1 as seen by Client 2.](https://s2.51cto.com/wyfs02/M02/8D/AE/wKiom1ilt5DD1uwkAACfpevkX3A342.png-wh_500x0-wm_3-wmp_4-s_4138866562.png)

Client 1 as seen by Client 2.

根据你所开发的游戏类型的不同，这个问题有很多不同的解决方案，你的游戏中的实体可预测性越高，你就越容易得出正确的结果。

Depending on the type of game you’re developing there are many ways to deal with this; in general, the more predictable your game entities are, the easier it is to get it right.

## Dead reckoning 航位推测法

假设你正在制作一个赛车游戏，一辆跑得非常快的车是很容易预测的，比如说，如果这辆车的速度是 100m/s ，一秒后这辆车的位置大概就是其前方100m处。

Suppose you’re making a car racing game. A car that goes really fast is pretty predictable – for example, if it’s running at 100 meters per second, a second later it will be roughly 100 meters ahead of where it started.

为什么是『大概』？因为在这一秒钟之间这两车可能加速或者减速了一点点，或者左拐右拐的一点点，关键词在于『一点点』。无论玩家做了什么操作，高速行驶的车辆的机动性很大程度上取决于它之前的位置，方向和速度。换而言之，一辆车不能瞬间180度大转弯～

Why “roughly”? During that second the car could have accelerated or decelerated a bit, or turned to the right or to the left a bit – the key word here is “a bit”. The maneuverability of a car is such that at high speeds its position at any point in time is highly dependent on its previous position, speed and direction, regardless of what the player actually does. In other words, a racing car can’t do a 180º turn instantly.

为什么这种方法适用于每100ms发送一次消息的服务器呢？客户端收到每一个对手车辆的已被验证过的速度和航向后，接下来的100ms之内并不会收到其他任何消息，但是客户端仍然需要将移动的过程显示出来，最简单的方式就是假设车辆的速度和航向在这100ms内保持不变，然后在本地移动车辆的物理位置。然后在100ms后，当服务器返回消息的时候再矫正车辆的位置。注意

How does this work with a server that sends updates every 100 ms? The client receives authoritative speed and heading for every competing car; for the next 100 ms it won’t receive any new information, but it still needs to show them running. The simplest thing to do is to assume the car’s heading and acceleration will remain constant during that 100 ms, and run the car physics locally with that parameters. Then, 100 ms later, when the server update arrives, the car’s position is corrected.

这里的「矫正」可能很大也可能很小，取决于许多的变量。如果玩家直直的开着车完全没有拐弯也没有踩刹车和油门，那么预测将会与最终位置完全匹配。然而如果玩家撞到什么东西的话，预测将会有巨大的误差。

The correction can be big or relatively small depending on a lot of factors. If the player does keep the car on a straight line and doesn’t change the car speed, the predicted position will be exactly like the corrected position. On the other hand, if the player crashes against something, the predicted position will be extremely wrong.

需要注意的是，航位推测法可以适用于低速情形，比如战列舰等。事实上「航位推测法」这个词本来就来自于海军导航术语。

Note that dead reckoning can be applied to low-speed situations – battleships, for example. In fact, the term “dead reckoning” has its origins in marine navigation.

## Entity interpolation 实体插值

有一些情形航标推测法是无法适用的，尤其是那些玩家的方向和速度会突然变化的那种。。比如说在3d射击游戏中，玩家经常会很以极快的速度走走停停转方向，由于位置和速度不再是可预测的，使得航标推测法完全没用了。

There are some situations where dead reckoning can’t be applied at all – in particular, all scenarios where the player’s direction and speed can change instantly. For example, in a 3D shooter, players usually run, stop, and turn corners at very high speeds, making dead reckoning essentially useless, as positions and speeds can no longer be predicted from previous data.

You can’t just update player positions when the server sends authoritative data; you’d get players who teleport short distances every 100 ms, making the game unplayable.

你无法仅当服务器下发数据的时候才更新位置，因为这样会导致玩家每100ms闪现一次，严重影响正常游戏～

你目前只有每100ms的玩家准确位置，因此关键在于如何在两个100ms之间把发生的事情「脑补」出来。解决这个问题只需要将其他玩家相对于玩家自己的“过去”的位置显示出来就好了。

What you do have is authoritative position data every 100 ms; the trick is how to show the player what happens inbetween. The key to the solution is to show the other players *in the past* relative to the user’s player.

假设你在 **t = 1000** 收到了所有玩家的位置信息，因为 **t = 900** 时的游戏状态已经收到了，所以你同时知道 **t = 900** 和 **t = 1000** 时的位置，所以从 **t = 1000** 到 **t = 1100** ，你可以显示其他玩家 **t = 900** 到 **t = 1000** 这段时间内发生的事情。如此你便一直显示着你自己的确切位置，而其他玩家则晚 100ms 才更新。

Say you receive position data at **t = 1000**. You already had received data at **t = 900**, so you know where the player was at **t = 900** and **t = 1000**. So, from **t = 1000** and **t = 1100**, you show what the other player did from **t = 900** to **t = 1000**. This way you’re always showing the user *actual movement data*, except you’re showing it 100 ms “late”.

![Client 2 renders Client 1 in the past, interpolating last known positions.](http://www.gabrielgambetta.com/img/fpm3-02.png)Client 2 renders Client 1 “in the past”, interpolating last known positions.

用来从**t = 900** 到 **t = 1000**插值的位置信息取决于游戏，一般来说插值法表现还不错。如果不行的话，你可以让服务器发送更多的细节给你，比如说发送玩家在移动过程中的位置序列，或者每10ms进行一次采样从而让插值结果更加自然。（也未必需要发送10倍的数据量，尝试改成发送增量式的移动可以在特定情况下可以得到优化）

The position data you use to interpolate from **t = 900** to **t = 1000** depends on the game. Interpolation usually works well enough. If it doesn’t, you can have the server send more detailed movement data with each update – for example, a sequence of straight segments followed by the player, or positions sampled every 10 ms which look better when interpolated (you don’t need to send 10 times more data – since you’re sending deltas for small movements, the format on the wire can be heavily optimized for this particular case).

需要注意的是使用这种技术，每个玩家可能看到的游戏世界略微不同。因为每个玩家都看到自己当前状态但是看到的是其他玩家的过去状态。即使是快节奏游戏中也是如此。然而一般来说100ms的延迟不是非常明显。

Note that using this technique, every player sees a slightly different rendering of the game world, because each player sees itself *in the present* but sees the other entities *in the past*. Even for a fast paced game, however, seeing other entities with a 100 ms isn’t generally noticeable.



There are exceptions – when you need a lot of spatial and temporal accuracy, such as when the player shoots at another player. Since the other players are seen in the past, you’re aiming with a 100 ms delay – that is, you’re shooting where your target was 100 ms ago! We’ll deal with this in the next article.

有一种例外，当你需要很精确的计算的时候，比如说一个玩家射击另一个玩家，因为你看到的是其他玩家是过去的位置，导致你总是延迟100ms才能瞄准，这就意味着你瞄准的总是100ms之前的目标，我们在下一篇文章中会处理这个问题。

## Summary 总结

即便是低频更新和网络延迟的情况下，你也必须让玩家感受到持续的平滑移动。在[第二篇文章](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html)中，我们讲解了客户端预测和服务器调和方法来让玩家实时移动，着确保玩家的输入会得到立即反馈，从而避免的延迟造成的严重问题。

In a client-server environment with an authoritative server, infrequent updates and network delay, you must still give players the illusion of continuity and smooth movement. In [part 2 of the series](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html) we explored a way to show the user controlled player’s movement in real time using client-side prediction and server reconciliation; this ensures user input has an immediate effect on the local player, removing a delay that would render the game unplayable.

但是其他的玩家控制的实体依然有问题，在本文中我们介绍了两种方法来解决。

Other entities are still a problem, however. In this article we explored two ways of dealing with them.

第一种是 *航位推测法* 在特定种类的模拟游戏中实体的位置可以根据当前的速度和加速度等等进行预测。在条件不符合的时候就会导致预测失败。

The first one, *dead reckoning*, applies to certain kinds of simulations where entity position can be acceptably estimated from previous entity data such as position, speed and acceleration. This approach fails when these conditions aren’t met.

第二种 *实体插值法* 完全不预测未来的位置，只使用实体的真实数据然后略微延迟其显示的时间。

The second one, *entity interpolation*, doesn’t predict future positions at all – it uses only real entity data provided by the server, thus showing the other entities slightly delayed in time.



The net effect is that the user’s player is seen *in the present* and the other entities are seen *in the past*. This usually creates an incredibly seamless experience.

However, if nothing else is done, the illusion breaks down when an event needs high spatial and temporal accuracy, such as shooting at a moving target: the position where Client 2 renders Client 1 doesn’t match the server’s nor Client 1′s position, so headshots become impossible! Since no game is complete without headshots, we’ll deal with this issue in the next article.

