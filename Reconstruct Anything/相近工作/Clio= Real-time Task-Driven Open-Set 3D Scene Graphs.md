---
id: Clio= Real-time Task-Driven Open-Set 3D Scene Graphs
tags:
  - 3D-Scene
  - Scene-graph
  - CLIP
  - Open-Vocabulary
date: 2025-03-18 16:54:35
categories: Review
publish:
---
![[Pasted image 20250318165935.png]]![[Pasted image 20250318165950.png]]

贡献：
- The first contribution of this paper is to propose a task-driven 3D scene understanding problem, where the robot is given a list of tasks in natural language, and has to select the granularity and the subset of objects and scene structure to retain in its map that is sufficient to complete the tasks.
- The second contribution is an algorithm for task-driven 3D scene understanding based on an Agglomerative IB approach, that is able to cluster 3D primitives in the environment into taskrelevant objects and regions
- 基于以上，实现了一个实时的pipeline

提出了**针对不同任务需要不同粒度的语义信息**，本文是通过结合SAM和[[CLIP多模态预训练模型]]实现，但是忽略了物体之间的谓语关系或者父子关系。本质还是智能做导航，拾取，放下，导航的基本操作。