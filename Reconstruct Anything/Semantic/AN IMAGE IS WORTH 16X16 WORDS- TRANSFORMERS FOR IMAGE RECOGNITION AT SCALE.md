---
id: AN IMAGE IS WORTH 16X16 WORDS- TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE
tags:
  - ViT
  - Transformer
date: 2025-01-09 16:18:40
categories: Note
---
![[Pasted image 20250109194820.png]]

https://www.youtube.com/watch?v=j3VNqtJUoz0&t=16s

核心思想：
- 将图像分为patches, 线性映射, 再加上图片的position embeding来输入transformer encoder
- 额外使用一个cls token用于占位（ViT的输出就是这个cls input token对应的output token）