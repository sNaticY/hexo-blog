---
title: 快节奏多人游戏同步(2)-客户端预测与服务器调和
date: 2018-05-07 17:37:11
tags: [multiplayer, sync, client, server]
categories: 快节奏多人游戏同步
---

[<< Series Start](http://www.gabrielgambetta.com/client-server-game-architecture.html)[Gabriel Gambetta](http://www.gabrielgambetta.com/index.html)

# Fast-Paced Multiplayer (Part II): Client-Side Prediction and Server Reconciliation

[Client-Server Game Architecture](http://www.gabrielgambetta.com/client-server-game-architecture.html) · [Client-Side Prediction and Server Reconciliation](http://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html) · [Entity Interpolation](http://www.gabrielgambetta.com/entity-interpolation.html) · [Lag Compensation](http://www.gabrielgambetta.com/lag-compensation.html) · [Live Demo](http://www.gabrielgambetta.com/client-side-prediction-live-demo.html)

## Introduction 概述

In the [first article](http://www.gabrielgambetta.com/client-server-game-architecture.html) of this series, we explored a client-server model with an authoritative server and dumb clients that just send inputs to the server and then render the updated game state when the server sends it.

在本系列的第一篇文章中，我们讨论了简单的权威服务器和傀儡客户端模型，即客户端发送输入到服务器，由服务器更新游戏状态后返回给客户端，最后再由客户端渲染的流程。

A naive implementation of this scheme leads to a delay between user commands and changes on the screen; for example, the player presses the right arrow key, and the character takes half a second before it starts moving. This is because the client input must first travel to the server, the server must process the input and calculate a new game state, and the updated game state must reach the client again.

该可能会流程导致用户输入和最终屏幕显示之间的延迟，例如玩家按下向右按键后半秒角色才开始移动，这是因为客户端的输入必须先发送到服务器并由服务器计算游戏状态并将结果发回客户端而导致的延迟。

![Effect of network delays.](http://www.gabrielgambetta.com/img/fpm2-01.png)Effect of network delays.

In a networked environment such as the internet, where delays can be in the orders of tenths of a second, a game may feel unresponsive at best, or in the worst case, be rendered unplayable. In this article, we’ll find ways to minimize or even eliminate that problem.

在非局域网环境下，延迟可能达到1/10秒，可能会导致游戏响应不够灵敏，最坏的情况可能导致基本玩不了。。在本文中，我们会找到减轻症状甚至解决这个问题的方案。

## Client-side prediction 客户端预测

Even though there are some cheating players, most of the time the game server is processing valid requests (from non-cheating clients and from cheating clients who aren’t cheating at that particular time). This means most of the input received will be valid and will update the game state as expected; that is, if your character is at (10, 10) and the right arrow key is pressed, it will end up at (11, 10).

纵使有一些作弊的玩家，但大多数情况下服务器都是在处理合法请求（包括非作弊玩家的请求和某些未在特定情况下作弊的客户端的请求）这意味着大多数输入将会是合法且游戏状态完全可预测。也就是说，如果你的角色在(10,10)位置，当按下向右按键时，该角色将会移动到(11,10)。

We can use this to our advantage. If the game world is *deterministic* enough (that is, given a game state and a set of inputs, the result is completely predictable),

如果游戏世界可预测性足够强（也就是说，一个给定的游戏状态加上一组输入，将可以得到固定的结果）的话，我们可以尝试利用这一点。首先假设我们的延迟是100ms，并且由一个格子移动到下一个格子的动画需要播放100ms，那么在最原始的实现中，整个角色的动作将延迟200ms

Let’s suppose we have a 100 ms lag, and the animation of the character moving from one square to the next takes 100 ms. Using the naive implementation, the whole action would take 200 ms:

因为游戏世界时可预测的，因此我们可以认为发到服务器的指令将会被立即执行，在这种情况下，客户端就可以预测输入执行以后的游戏状态了，而且大多数情况下这个预测都是基本正确的。

![Network delay + animation.](http://www.gabrielgambetta.com/img/fpm2-02.png)Network delay + animation.

Since the world is deterministic, we can assume the inputs we send to the server will be executed successfully. Under this assumption, the client can predict the state of the game world after the inputs are processed, and most of the time this will be correct.

因此我们可以发送输入后在等待服务器返回的同时立刻渲染输入执行成功后的结果，而不是傻等着服务器返回结果后再进行渲染。这样大多数情况下客户端自行计算的结果基本上是与服务器的返回相匹配的。

Instead of sending the inputs and waiting for the new game state to start rendering it, we can send the input and start rendering the outcome of that inputs as if they had succeded, while we wait for the server to send the “true” game state – which more often than not, will match the state calculated locally :

![Animation plays while the server confirms the action.](http://www.gabrielgambetta.com/img/fpm2-03.png)Animation plays while the server confirms the action.

所以现在在玩家输入与屏幕渲染之间完全没有延迟了，而且游戏服务器依然是权威服务器。因为当被破解的客户端发送无效请求时虽然会在自己的屏幕上显示出来，但并不会影响到其他玩家。

Now there’s absolutely no delay between the player’s actions and the results on the screen, while the server is still authoritative (if a hacked client would send invalid inputs, it could render whatever it wanted on the screen, but it wouldn’t affect the state of the server, which is what the other players see).



## Synchronization issues 同步问题

In the example above, I chose the numbers carefully to make everything work fine. However, consider a slightly modified scenario: let’s say we have a 250 ms lag to the server, and moving from a square to the next takes 100 ms. Let’s also say the player presses the right key 2 times in a row, trying to move 2 squares to the right.

在上面的例子中，我们刚好选了一个可以让一切正常运行的数值，然而稍微考虑一下以下场景，客户端到服务器的延迟是250ms，但是从一个格子移动到另一个格子只需要100ms，并且玩家尝试两次按下向右按键并向右移动两格。

Using the techniques so far, this is what would happen:

按照目前的实现，我们将看到如下状况。

![Predicted state and authoritative state mismatch.](http://www.gabrielgambetta.com/img/fpm2-04.png)Predicted state and authoritative state mismatch.

我们在 **t = 250 ms** 的时候面临一个非常 interesting 的问题，当新的游戏状态回来时，客户端预测的位置已经到达 **x = 12**，但是服务器认为最新的坐标是 **x = 11**，因为权威服务器的缘故，客户端必须将角色移回 **x = 11**，但是紧接着，新的 **x = 12** 的状态在 **t = 350** 的时间到达，因此角色的位置又顺移回去了。。

We run into an interesting problem at **t = 250 ms**, when the new game state arrives. The predicted state at the client is **x = 12**, but the server says the new game state is **x = 11**. Because the server is authoritative, the client must move the character back to **x = 11**. But then, a new server state arrives at **t = 350**, which says **x = 12**, so the character jumps again, forward this time.

From the point of view of the player, he pressed the right arrow key twice; the character moved two squares to the right, stood there for 50 ms, jumped one square to the left, stood there for 100 ms, and jumped one square to the right. This, of course, is unacceptable.

从玩家的角度来看，他按下两次向右按钮后，角色向右移动两格，原地停留50ms后，向左顺移一格，又原地停留100ms再向右顺移一个，很明显这种情况是不可接受的。

## Server reconciliation 服务器调和

The key to fix this problem is to realize that the client sees the game world *in present time*, but because of lag, the updates it gets from the server are actually the state of the game *in the past*. By the time the server sent the updated game state, it hadn’t processed all the commands sent by the client.

问题的关键在于，客户端显示的是*当前时间*的游戏状态，但是因为延迟的关系，收到来自服务器的回复是*过去*的游戏状态。因为在收到服务器更新的时间点服务器并没有处理客户端发送的全部输入。

This isn’t terribly difficult to work around, though. First, the client adds a sequence number to each request; in our example, the first key press is request #1, and the second key press is request #2. Then, when the server replies, it includes the sequence number of the last input it processed:

这并不是一个非常严重的问题，首先，客户端在每次请求的时候加上 sequence number，在我们的例子中，第一次按键指令编号为 #1，第二次按键的指令编号为 #2。服务器回复的时候将其处理过最后一个 sequence number 包含在消息中。

![Client-side prediction + server reconciliation.](http://www.gabrielgambetta.com/img/fpm2-05.png)Client-side prediction + server reconciliation. 

那么现在，在 **t = 250** 的时候，服务器回复说 “**基于#1号请求的结果，你的坐标是x=11**”，因为权威服务器的关系，角色坐标将会被设置为**x = 11**。如果客户端将每一个发送至服务器的指令都保存下来的话，当收到了服务器的回包，客户端就知道服务器已经执行了 #1 号请求，所以就可以将保存下来的#1号请求的备份删除掉了，并且客户端还知道服务器尚未回复#2号请求的更新结果，因此客户端就可以基于服务器已经认证过的上次请求的结果结合服务器尚未处理的请求来重新计算客户端预测的状态。

Now, at **t = 250**, the server says “**based on what I’ve seen up to your request #1, your position is x = 11**”. Because the server is authoritative, it sets the character position at **x = 11**. Now let’s assume the client keeps a copy of the requests it sends to the server. Based on the new game state, it knows the server has already processed request #1, so it can discard that copy. But it also knows the server still has to send back the result of processing request #2. So applying client-side prediction again, the client can calculate the “present” state of the game based on the last authoritative state sent by the server, plus the inputs the server hasn’t processed yet.

所以在 **t = 250** 的时候，客户端收到 “**x = 11, 上次处理的请求 = #1**”。于是客户端丢弃了#1号请求的备份，因为服务器尚未处理#2号请求，因此客户端依然保留#2号请求的备份。该行为将会使得客户端基于服务器发送的 **x = 11** 来更新其内部游戏状态，然后将服务器尚未接收到的请求进行模拟。于是 #2号请求“向右移动”将会得到正确的结果，**x = 12**。

So, at **t = 250**, the client gets “**x = 11, last processed request = #1**”. It discards its copies of sent input up to #1 – but it retains a copy of #2, which hasn’t been acknowledged by the server. It updates it internal game state with what the server sent, **x = 11**, and then applies all the input still not seen by the server – in this case, input #2, “move to the right”. The end result is **x = 12**, which is correct.

继续我们的例子，在 **t = 350** 的时候，从服务器收到了新的游戏状态 “**x = 12, 上次处理的请求 = #2**”。于是客户端丢弃了 #2号请求前的所有备份，并将游戏状态更新到了**x = 12**。目前已经没有尚未被处理的请求了，进程结束在正确的状态。

Continuing with our example, at **t = 350** a new game state arrives from the server; this time it says “**x = 12, last processed request = #2**”. At this point, the client discards all input up to #2, and updates the state with **x = 12**. There’s no unprocessed input to replay, so processing ends there, with the correct result.

## Odds and ends 偏差与结果

虽然以上的例子只讨论了移动的情况，但是同样地规则可以适用于几乎任何状况，例如在回合制对抗游戏中，当玩家攻击其他的角色时，可以先显示血量和数值就好像已经完成攻击了一样，但是直到服务器回复之前不可以直接更新角色的血量。

The example discussed above implies movement, but the same principle can be applied to almost anything else. For example, in a turn-based combat game, when the player attacks another character, you can show blood and a number representing the damage done, but you shouldn’t actually update the health of the character until the server says so.


Because of the complexities of game state, which isn’t always easily reversible, you may want to avoid killing a character until the server says so, even if its health dropped below zero in the client’s game state (what if the other character used a first-aid kit just before receiving your deadly attack, but the server hasn’t told you yet?)

This brings us to an interesting point – even if the world is completely deterministic and no clients cheat at all, it’s still possible that the state predicted by the client and the state sent by the server don’t match after a reconciliation. The scenario is impossible as described above with a single player, but it’s easy to run into when several players are connected to the server at once. This will be the topic of the next article.

### Summary

When using an authoritative server, you need to give the player the illusion of responsiveness, while you wait for the server to actually process your inputs. To do this, the client simulates the results of the inputs. When the updated server state arrives, the predicted client state is recomputed from the updated state and the inputs the client sent but the server hasn’t acknowledged yet.
