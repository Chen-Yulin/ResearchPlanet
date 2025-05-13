---
id: RelTR= Relation Transformer for Scene Graph  Generation
tags:
  - Visual-Relation
  - Transformer
  - Scene-graph
date: 2025-03-14 12:38:41
categories: Review
publish: 2023 Apr
---
![[Pasted image 20250314123930.png]]

RelTR是自下而上的方法, 使用基于Transformer的 object detector（例如DETR）生成对象候选者，然后使用relation transformer来预测object pairs之间的关​​系。它还设计了一种基于积分的关系表示方法，该方法将关系编码为二维矢量场。

