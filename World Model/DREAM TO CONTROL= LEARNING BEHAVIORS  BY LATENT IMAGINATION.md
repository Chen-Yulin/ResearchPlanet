---
id: DREAM TO CONTROL= LEARNING BEHAVIORS  BY LATENT IMAGINATION
tags:
  - Embodied-AI
  - WorldModel
  - RL
date: 2025-11-22 16:10:20
categories: Review
publish: "2020"
---
![[Pasted image 20251122161127.png]]

## 框架
论文使用**RSSM(Recurrent State Space Model)**：使用encoder来编码环境和动作生成latent state, 预测未来latent state，最后基于latent state预测奖励。

优势：
- 网络可以在 latent 中快速 roll-out 数千条 imagined trajectories
- 不用预测 pixel → 速度极快
- 潜在空间的 Markov 性保证了规划时的可微分性

## 重参数化 Reparameterization Trick

Dreamer 最关键的地方：

> **动作必须是可微的随机变量，这样梯度才能从 value 反传到 actor。**

如果我们直接写 $a \sim \mathcal{N}(\mu, \sigma)$, 那么采样是不可微的 → 梯度断掉 → Actor 无法学习。
重参数化技巧的做法：$a=\mu+\sigma\cdot\epsilon$, $\epsilon\sim\mathcal{N}(0,1)$
现在：
- ε 是随机的
- μ 和 σ 是可微的网络输出
所以动作对 actor 参数有梯度, 这就是可微规划（differentiable planning）的基础。