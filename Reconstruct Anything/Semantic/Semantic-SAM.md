---
id: Semantic-SAM
tags:
  - Segmentation
  - Semantic
date: 2025-03-06 14:27:14
categories: Review
publish: 2023 July
---
![[Pasted image 20250306152156.png]]

![[Pasted image 20250307140415.png]]

![[Pasted image 20250306142804.png]]

这片文章可以成为场景物理重建的基石之一
类似的后续工作有OMG-Seg

## 是什么
通用图像分割模型，以实现细分并识别任何所需粒度的任何内容
- Semantic-awareness
- Granularity-abundance

## 数据集
![[Pasted image 20250310201828.png]]
### 难点
- 目前的一些通用的物体和分割数据集虽然确实提供了大体量的数据和丰富的语义信息，但只局限在object level。
- 目前一些分割了细分part的数据集却体量有限。
- SAM使用的数据集为多粒度的大体量数据集，但是并不包含语义标注。
### 解决方式
合并了多个不同分割粒度的数据集











