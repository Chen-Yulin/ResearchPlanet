---
id: Deformable Convolutional Networks
tags:
  - CV
  - NN
date: 2025-05-06 11:55:36
categories: Review
publish: "2017"
---
Used in [[CenterNet]]

pre: https://www.youtube.com/watch?v=HRLMSrxw2To&t=308s

## 解决的问题
Modeling spatial transformations is a long standing problem in computer vision
- Deformation (human pose)
- Scale
- Viewpoint variation
- Intra-class variation (不同设计的同一种物体)

Traditional approaches:
- build datasets with sufficient desired variations
- use transformation-invariant features and algorithms

## 架构
![[Pasted image 20250506120853.png]]

![[Pasted image 20250506120920.png]]

## 优势
与传统CNN拥有相同的输入输出
- regular convolution -> deformable convolution
- regular RoI pooling -> deformable RoI pooling

可以端到端训练且无需额外监督信号

直接认为是一种在物体检测方面即插即用的模块即可
