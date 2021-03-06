---
title: "第11.0讲：时间差分学习"
layout: page
date: 2018-03-30
---
[TOC]

## 关于
强化学习近年大火，最早是因为AlphaGo使用强化学习打败人类围棋冠军引发的。在那之后，强化学习在工业场景应用越来越多，原来很多做搜索、推荐、广告等一直在用监督学习的业务，也开始使用强化学习来优化用户体验和平台收益了。强化学习实际上很早就提出了，事实上，强化学习来源于控制论。控制论之父叫做[维纳](https://en.wikipedia.org/wiki/Norbert_Wiener)，学过信号处理的可能知道他，维纳滤波就是用他的名字命名的。控制论最早源于航天，人们要控制火箭发射装置，将火箭发射到地球之外；控制论还源于机器人控制，人们需要用算法对机器人的行为进行控制。所以，很多讲强化学习的文献也会说强化学习是在优化控制策略。

在这篇文章中，您将学习到

1. 环境未知情况下的更好的学习方法？
2. 蒙特卡罗方法和时间差分学习
3. SARSA, Q-learning
4. on-policy 与 off-policy 学习

我期望您至少有：

1. 高中数学水平且年满18岁，部分内容需要你了解监督学习，和强化学习基本概念，你可以通过本教程前面的章节进行学习。
2. 如果你需要完成实践部分，需要有基本的 python 知识，你可以通过[python快速入门](/wiki/tutorial/ml/intro-python.html)快速了解python如何使用。

## 蒙特卡罗方法
在上一讲中，我们说到，对于一个MDP问题，如果环境已知，也就是知道环境的转移概率P和回报函数R，可以通过求解贝尔曼方程得到值函数，从而得到最优策略，求解的方法有两种，分别是值迭代和策略迭代。如果环境未知，我们提到一种通过随机尝试的方法，先估计出环境，然后转化为环境已知问题求解。这种随机尝试效率很差，需要尝试很多次，并且随机尝试本身就有成本，而且需要等到尝试很多次之后，才能得到一个比较好的策略。那么，能不能在每一次尝试之后，都能将策略提升到一个更好的策略呢？因为每一次尝试，环境的反馈都会提供一部分对环境的信息，如果我们能利用好这部分信息，就有可能从中获取到有价值的信息来提升我们的策略。

还是以走迷宫为例（如图所示），并且考虑确定性迷宫这种简单情况。我们的目标是估计出每个状态s下，采取每一个动作a的动作值函数Q，即从s出发，第一步采取动作a所能获得的最大回报。那么，一个简单的想法是，从某个状态s出发，例如状态1，如果某一次行动，采用动作a=向右获得了钻石，那么动作a=向右更有可能有较大的Q；相反如果a=向下掉到了火坑，那么动作a=向下更有可能有较小的Q。这表明，从某一次的尝试中，已经能够获得一些关于环境的信息，虽然信息量没有多到足以完全确定最优策略的地步，但是这个信息已经可以用来更新我们的策略了。例如，在以后的尝试中，可以给动作a=向右以更大的概率，而给动作a=向下以更小的概率。

![迷宫任务](/wiki/static/images/mdp-01.png)


> **为什么估计动作值函数而不是状态值函数？**
>
> 我们的目标是得到动作值函数Q(s, a)而不是状态值函数V(s)，因为只有状态值函数V(s)的情况下，我们还需要知道转移概率P才能得到动作值函数，反过来则简单得多。在环境未知，也就是转移概率你无法知道，所以要估计动作值函数

如果我们从状态s出发，采用动作a尝试了n次，每一次的总回报为$G_i$，那么容易估计出期望回报

$$
Q_n(s, a) = \frac{1}{n}\sum_{i=1}^n G_i
$$

想象一下，这n次行动是一次执行的，上式需要这n次尝试完全结束后，才能估计Q，事实上可以把上式改写为递归形式

$$
Q_n(s, a) = \frac{1}{n}(G_n + (n-1) Q_{n-1}(s, a)) \\\\
= Q_{n-1}(s, a) + \frac{1}{n}\left( G_n - Q_{n-1}(s, a) \right)
$$

也就说，每一次尝试实际得到的回报可以用来调整之前对Q函数的估计，每一次对Q函数的改变就是上一次估计的误差$G_n - Q_{n-1}(s, a)$乘上一个系数$\alpha = \frac{1}{n}$。

这种利用经验从未知环境中估计值函数的方法叫做蒙特卡罗法。

> ** 梯度下降 **
>
