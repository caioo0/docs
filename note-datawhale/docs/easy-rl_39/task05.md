# task05:稀疏奖赏和模仿学习

> （本学习笔记来源于DataWhale-第39期组队学习：[强化学习](https://linklearner.com/datawhale-homepage/#/learn/detail/91)） ,
> [B站视频讲解](https://www.bilibili.com/video/BV1HZ4y1v7eX) 观看地址


## 稀疏奖赏

---

### 什么是稀疏奖赏？

稀疏奖赏指对于某些环境而言，在强化学习的训练过程中大部分时候都无法获得奖励，使得agent难以学到动作或策略的价值。在这类系统中完成RL的训练是困难的，由于奖赏的稀疏或延迟，多数情况下动作缺乏引导和评估，因而很难进行学习。为解决这类问题，人们提出了不同的方案，其中比较常见的有：奖励函数设计（Reward Shaping）、课程学习（Curriculum Learning）、分层强化学习（Hierarchical RL）等。

### 什么是奖赏函数设计（Reward Shaping）？

奖赏函数设计最容易理解，就是通过人为地在环境中设置一些奖赏，来引导agent的学习。比如射击类游戏中，原本需要最终击杀敌人才能获得奖励，但在此过程中还有许多必要或重要的步骤，在没有奖励的情况下agent很难在短时间内学习到正确的流程，因此需要在过程中对特定状态-动作（如转身搜索目标、换弹夹或寻找掩体等）设计一些奖赏，来引导其更快更好地向最终目标进发。奖赏函数设计可以一定程度上解决稀疏奖赏的问题，但需要设计者对于环境有充分的了解，否则难以设计出合适的奖赏函数。

### 什么是好奇心机制（Curiosity）？

奖赏函数设计方法中有一类较为独特的方法被称为好奇心机制。在好奇心机制中，首先引入一套新的被称为ICM(intrinsic curiosity module) 的奖励函数，它的奖励**\(r_i\)**同原始的奖赏一样会被加入到总的累积奖赏中，这套奖励函数运行的机制如下：首先有一个神经网络，它在RL训练过程中不断输入当前的状态**\(s_t\)**和动作**\(a_t\)**，输出是预测的下一个动作**\(s_{t+1}\)**。当输出的预测值与实际的下一状态差别越大，则所获的奖励**\(r_t\)**就越大。也就是说好奇心机制会自行驱动agent去探索没有见到过的状态，或采取过去未曾用过的动作，而不会因为奖赏稀疏就在已探索过的范围内打转。通过这样一套奖励函数的引入可以较好地引导agent在稀疏奖励下的探索。

### 什么是课程学习（Curriculum Learning）？

参考资料：

1. DataWhale组队学习资料——《强化学习》 王琦 杨毅远 江季 著
