---
id: Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling
tags:
  - NLP
  - RNN
  - Gated-NN
date: 2024-09-27 12:56:52
categories: Review
aliases:
  - Gated-RNN
---
## Background: RNN
首先介绍了RNN通过hidden state来实现记忆力功能
![[Pasted image 20240927130036.png]]
但指出RNN的训练有梯度消失/爆炸的现象，且记忆会沿序列长度的增加而指数下降，缺乏长期记忆能力。
解决梯度消失/爆炸目前有梯度裁剪和二阶梯度的方法，但成效并不显著

## Gated RNN
[[On the Properties of Neural Machine Translation= Encoder–Decoder Approaches]]
