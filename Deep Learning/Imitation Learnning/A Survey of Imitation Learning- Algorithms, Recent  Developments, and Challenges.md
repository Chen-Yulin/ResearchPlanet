---
id: A Survey of Imitation Learning- Algorithms, Recent  Developments, and Challenges
tags:
  - Survey
  - ImitationLearning
date: 2024-12-31 13:13:32
categories: Note
---
## Introduction
IL是区别于传统手动编程来赋予机器人自主能力的方法。
IL 允许机器通过演示（人类演示专家行为）来学习所需的行为，从而消除了对显式编程或特定于任务的奖励函数的需要。
IL主要有两个类别：
- 行为克隆(BC)
- 反向强化学习(IRL)
![[Pasted image 20241231131825.png]]

## Behavior Cloning
BC 是一种 IL 技术，它将学习行为的问题视为监督学习任务 。 BC 涉及通过建立环境状态与相应专家操作之间的映射来训练模型来复制专家的行为。专家的行为被记录为一组state-action pair，也称为演示。在训练过程中，模型学习一个函数，利用这些演示作为输入，将当前状态转换为相应的专家操作。经过训练，模型可以利用这个学习函数来生成遇到新状态的动作。

不需要了解环境的潜在动态，计算效率很高，相对简单的方法。

The covariate shift problem: 测试期间观察到的状态分布可能与训练期间观察到的状态分布有所不同，使得代理在遇到未见过的状态时容易出错，而对于如何进行操作缺乏明确的指导。BC监督方法的问题是，当智能体漂移并遇到分布外状态时，它不知道如何返回到演示的状态。

为了解决这个问题：
![[Pasted image 20241231133031.png]]


## Inverse Reinforcement Learning
IRL 涉及一个学徒代理，其任务是推断观察到的演示背后的奖励函数，这些演示被认为源自表现最佳的专家 。然后使用推断的奖励函数通过 RL 训练学习代理的策略。

为了解决“政策->奖励函数“的模糊性，有以下三种IRL
- maximum-margin methods（奖励函数比任何其他策略在一定程度上更全面地解释最优策略。这本质上意味着找到一个最大化指定利润的解决方案，确保派生的奖励函数捕捉专家行为的本质。）
- maximum entropy（处理专家次优性和随机性的有前景的能力）
- guided cost learning（旨在优化策略优化内循环内的非线性奖励函数的方法。这种方法通过直接利用系统的原始状态来构建奖励函数，从而改变了传统的 IRL 范式，从而消除了广泛的特征工程的需要。）

## Adversarial Imitation Learning
The agent strives to deceive the discriminator by generating trajectories closely resembling those of the expert.

## Imitation From Observation
仅通过图像序列来学习，不需要具体的关节动作操作数据。
> Unlike the traditional methods, IfO presents a more organic approach to learning from experts, mirroring how humans and animals approach imitation. Humans often learn new behaviors by observing others without detailed knowledge of their actions (e.g., the muscle commands). People learn a diverse range of tasks, from weaving to swimming to playing games, by watching online videos. Despite differences in body shapes, sensory inputs, and timing, humans exhibit an impressive ability to apply knowledge gained from the online demonstrations

将可学习的资源扩大到了线上的视频资源。

### Latent Action Policies (LAPOs)
过分析观察到的动态，LAPO 推断出行动空间的底层结构，促进潜在行动策略的训练。然后，这些策略可以进行高效的微调，以达到专家级的性能，从而提供离线和在线场景的适应性。使用包含标记动作的小数据集进行离线微调是可行的，而在线微调可以使用奖励来完成。与依赖标记数据来训练逆动力学模型不同，LAPO**直接从观察到的环境动态中导出潜在动作信息，而不需要任何标签**。


## Challenges And Limitations
。。。




