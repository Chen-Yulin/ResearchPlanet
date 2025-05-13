---
id: Attention Is All You Need
tags:
  - Transformer
  - NLP
date: 2024-09-27 12:56:33
categories: Review
aliases:
  - Transformer
---
## 概要 
Transformer是一种基于注意力机制，完全不需要递归或卷积网络的序列预测模型，且更易于训练

## 背景
介绍了Gated-RNN/LSTM的基本逻辑[[Empirical Evaluation of Gated Recurrent Neural Networks on Sequence Modeling]]，指出:
这种固有的顺序性质阻碍了训练示例中的并行化，这在较长的序列长度上变得至关重要，因为内存限制限制了示例之间的批处理，虽然后续有相关工作优化了一些性能，但是基本的限制并没有解除。

## 代码
https://github.com/hkproj/pytorch-transformer/
https://www.youtube.com/watch?v=ISNdQcPhsts
![[Pasted image 20250216164647.png]]
![[Pasted image 20250216164655.png]]
![[Pasted image 20250216164703.png]]
![[Pasted image 20250216164709.png]]

关于Transformer 中的LayerNorm: https://dkleine.substack.com/p/understanding-layer-normalization?utm_campaign=post&utm_medium=web

![[Pasted image 20250505184208.png]]