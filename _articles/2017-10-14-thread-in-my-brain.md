---
title: "开车会占用 3 个线程"
date: 2017-10-14 19:07:17 +0800
categories: mind
tags: 大脑 线程
---

某天早上醒来，我的大脑中央处理核心已经升级成双核四线程了。这意味着我可以同时做两件完全不同的事情，可以同时做四件相关的事情。

---

在发现这项神奇的系统更新之后，我感觉每天过得更加有效率了。不过好景不长，最近接二连三发生一些令我极度沮丧的事情。

正常情况下，总会有线程在空闲时不断接受外界的输入，各种线程都可以。但是“沮丧”的信息源却是储存在大脑记忆体中的，从记忆体中不断发出输入信号，频率非常高，以至于必须一直有一个线程来处理这个输入。由于这种输入采用硬件中断的方式，没有任何办法能够忽略这样的输入信号源。就这样，其中一个线程全天处于处理“沮丧”的信号输入。

这还没完！正常的信息本应该是纯数据的，但“沮丧”信息却会从数据中解析出可执行逻辑。在处理此信息输入时，必须向此可执行逻辑传入一个委托。“沮丧”逻辑执行时会不断在新线程中调用这个委托，以便在脑海中形成一段一段支离破碎的情节。这些情节无法组成一个完整的故事，因为生成故事所需的算法需要在另一个新的线程执行，否则线程饿死概率会较高从而无法生成完整的故事。于是，我放弃了故事生成算法。

---

今天开车。开车是一个复杂的可执行逻辑，需要两个线程同时执行。一个线程负责收集外界环境信息，包括道路、红绿灯、车流量等；另一个线程负责计算路线和速度以便适应这样的外界环境。

为了确保安全，今年三月份我习得了一项新的算法——[防御性驾驶算法](https://www.zhihu.com/question/28878219)。这个算法的本质是——假想道路上的一切活体都是傻 X，他们无时无刻不想撞你或被你撞；为了避免碰撞，而开始进行更远处的活体检测，并且对每一个活体行为使用不断训练的神经网络进行预测。可想而知这个新算法是多么地耗资源，好在一个线程就足够计算了。

可是，我已经有两个线程处于无法忽略地持续处理记忆体内在信息源了，只剩下两个空闲线程。开车时，这两个线程当然得分配给开车所需的最基础资源需求中。那么“防御性驾驶算法”如何才能有效地执行！

---

虽说开车时这四个线程都处于活动状态，但其实微小处也是能插入一些小的可执行片段的，毕竟我有 `Yield` 神器（阅读 [出让执行权：Task.Yield, Dispathcer.Yield](/post/yield-in-task-dispatcher) 以了解更多）。

于是，我改造了防御性驾驶算法，均匀地将其可执行片段分洒在各个不同的线程中。

我感觉到防御性驾驶正在起作用，即便是四个线程都处在活跃状态下也依然正在工作，工作得很正常，正很常。车一边我开，乱一边思胡想。

车开着，开着，屏幕上逐渐浮现起一些诡异的故事，然而在改造算法的时候似乎并没有设计这样的逻辑代码啊！令我感到惊奇的是，每次开车都有不一样的故事。我正在写下这篇故事，哦不，我正在开车，哦不，这篇故事是我正在开车的一段旅程。

车到了，故事写完了。

我发现改造后的算法效率真高，除了额外多执行了本需要一整个额外线程才能执行的任务之外，还执行了一些我也不知道是什么的代码。但我不管，这效果有点像四核八线程的中央处理核心。赚了！

---

昨天小伙伴在坐我开的车时，不断地与我说话，这样我不得不放弃执行“线程改良版防御性驾驶算法”。虽说说话耗的资源比较少，但毕竟本就塞得满满的了，再也塞不下新的算法了。
