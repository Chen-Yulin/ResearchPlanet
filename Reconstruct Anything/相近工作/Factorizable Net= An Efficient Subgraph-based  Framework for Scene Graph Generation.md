---
id: Factorizable Net= An Efficient Subgraph-based  Framework for Scene Graph Generation
tags:
  - Scene-graph
  - Subgraph
  - Visual-Relation
date: 2025-03-16 16:46:04
categories: Review
publish: "2018"
---
![[Pasted image 20250316164807.png]]
The **extensibility** and **inference speed** of a SGG framework is crucial for accelerating down-stream tasks. This paper studied the efficiency and scalability in SGG
## Insights
最大的亮点是将相似的相互作用区域的对象对聚集到子图中并共享短语表示（称为子图特征），然后再在子图上refine

> 我的想法是将场景进行panoptic segmentation 之后再在每个物体上进行hierarchical part relation detection，异曲同工。