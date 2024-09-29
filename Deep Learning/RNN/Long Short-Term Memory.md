---
id: LSTM
tags:
  - RNN
  - NLP
  - Gated-NN
date: 2024-09-29 12:22:55
categories: Review
aliases:
  - LSTM
---
## 背景
LSTM主要是用于解决递归网络中梯度指数级消失或者梯度爆炸的问题

https://www.youtube.com/watch?v=YCzL96nL7j0&t=267s
LSTM和RNN主要的区别就在于：LSTM有两条记忆链，一条短期记忆，一条长期记忆。
![[Image_1727583705949.png]]
主要分成三个模块
- Forget Gate: 决定遗忘多少长期记忆
- Input Gate: 决定将多少当前输入存入长期记忆
- Output Gate: 基于短期记忆和输入决定输出的百分比，乘上长期记忆激活后的值，获得新的短期记忆，也就是输出。

![[Image_1727583734604.png]]
这里gate的概念启发了grConv[[On the Properties of Neural Machine Translation= Encoder–Decoder Approaches]]