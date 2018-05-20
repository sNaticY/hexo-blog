---
title: 快节奏多人游戏同步(4)-延时补偿
date: 2018-05-07 17:39:41
tags: [multiplayer, sync, client, server]
categories: 快节奏多人游戏同步
---

[<< Series Start](http://www.gabrielgambetta.com/client-server-game-architecture.html)[Gabriel Gambetta](http://www.gabrielgambetta.com/index.html)

# Fast-Paced Multiplayer (Part IV): Lag Compensation

[Client-Server Game Architecture](http://www.gabrielgambetta.com/client-server-game-architecture.html) · [Client-Side Prediction and Server Reconciliation](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html) · [Entity Interpolation](http://www.gabrielgambetta.com/entity-interpolation.html) · [Lag Compensation](http://www.gabrielgambetta.com/lag-compensation.html) · [Live Demo](http://www.gabrielgambetta.com/client-side-prediction-live-demo.html)

## Introduction 概述

The previous three articles explained a client-server game architecture which can be summarized as follows:

之前三篇文章主要解释了关于 client-server 架构的游戏的以下三个结论

- Server gets inputs from all the clients, with timestamps
- 服务器从所有的客户端获取带着时间戳的输入
- Server processes inputs and updates world status
- 服务器负责处理所有输入并更新游戏世界的状态
- Server sends regular world snapshots to all clients
- 服务器向所有客户端发送游戏世界的快照
- Client sends input and simulates their effects locally
- 客户端发送输入并自己模拟该输入造成的结果
- Client get world updates and
- 客户端获取世界更新并
  - Syncs predicted state to authoritative state
  - 同步
  - Interpolates known past states for other entities
  - 将客户端自行预测的状态更新到已校验的状态
  - 将其他客户端控制的实体插值到过去的状态

From a player’s point of view, this has two important consequences:

从玩家的角度来看，以上方案会产生两个重要结果：

- Player sees **himself** in the **present** 
- 玩家看到 **自己** 处于 **现在**

- Player sees **other entities** in the **past** 
- 玩家看到 **其他玩家** 处于 **过去**

This situation is generally fine, but it’s quite problematic for very time- and space-sensitive events; for example, shooting your enemy in the head!

通常来说这种情况并没有什么问题，但是在时间或空间敏感的状况下不太适合，比如说爆头

## Lag Compensation 延时补偿

假设你正用狙击枪完美的瞄准目标的头部，此时射击绝对万无一失。

So you’re aiming perfectly at the target’s head with your sniper rifle. You shoot - it’s a shot you can’t miss.

然而却没打到。。。

But you miss.

为什么会发生这种事情。。

Why does this happen?

因为我们之前解释过的 client-server 架构，你瞄准的是 100ms 之前的玩家的头，而不是开枪的时候的玩家的头。。。

Because of the client-server architecture explained before, you were aiming at where the enemy’s head was 100ms *before* you shot - *not*when you shot!

在某种程度上，相当于在宇宙中光速非常非常慢，你瞄准的是敌人过去的位置，但你扣下扳机的时候他早就走远了。。

In a way, it’s like playing in an universe where the speed of light is really, really slow; you’re aiming at the past position of your enemy, but he’s long gone by the time you squeeze the trigger.

幸运的是，有一个相对简单的方案可以让大多数玩家在大多数情况下满意（下面会解释）

Fortunately, there’s a relatively simple solution for this, which is also pleasant for *most* players *most* of the time (with the one exception discussed below).

是这样：

Here’s how it works:

- When you shoot, client sends this event to the server with full information: the exact timestamp of your shot, and the exact aim of the weapon.
- 开火的时候，客户端发送开火指令到服务器，同时包含开火的一瞬间确切的时间和方向。
- **Here’s the crucial step**. Since the server gets all the input with timestamps, it can authoritatively reconstruct the world at any instant in the past. In particular, it can reconstruct the world exactly as it looked like to any client at any point in time.
- **至关重要的一步**，服务器获取到所有带有时间戳的输入后，服务器可以重新构建过去任何时刻的游戏世界。尤其是可以构建任何客户端在任何时间点看到的几乎完全一致的世界
- This means the server can know exactly what was on your weapon’s sights the instant you shot. It was the *past* position of your enemy’s head, but the server knows it was the position of his head in *your* present.
- The server processes the shot *at that point in time*, and updates the clients.

And everyone is happy!

The server is happy because he’s the server. He’s always happy.

You’re happy because you were aiming at your enemy’s head, shot, and got a rewarding headshot!

The enemy may be the only one not entirely happy. If he was standing still when he got shot, it’s his fault, right? If he was moving… wow, you’re a really awesome sniper.

But what if he was in an open position, got behind a wall, and *then* got shot, a fraction of a second later, when he thought he was safe?

Well, that can happen. That’s the tradeoff you make. Because you shoot at him in the past, he may still be shot up to a few milliseconds after he took cover.

It is somewhat unfair, but it’s the most agreeable solution for everyone involved. It would be much worse to miss an unmissable shot!

## Conclusion

This ends my series on Fast-paced Multiplayer. This kind of thing is clearly tricky to get right, but with a clear conceptual understanding about what’s going on, it’s not exceedingly difficult.

Although the audience of these articles were game developers, it found another group of interested readers: gamers! From a gamer point of view it’s also interesting to understand why some things happen the way they happen.

### Further Reading

As clever as these techniques are, I can’t claim any credit for them; these articles are just an easy to understand guide to some concepts I’ve learned from other sources, including articles and source code, and some experimentation.

The most relevant articles about this topic are [What Every Programmer Needs to Know About Game Networking](http://gafferongames.com/networking-for-game-programmers/what-every-programmer-needs-to-know-about-game-networking/) and [Latency Compensating Methods in Client/Server In-game Protocol Design and Optimization](https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization).
