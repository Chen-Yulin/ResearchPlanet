---
id: Fully Convolutional Scene Graph Generation
tags:
  - Visual-Relation
  - Scene-graph
  - CV
  - FPN
date: 2025-03-14 12:29:24
categories: Review
publish: "2021"
---
![[Pasted image 20250314123239.png]]
One stage relation detection (detects objects and relations simultaneously).

这个模型受启发于 [[CenterNet]] 和 [[OpenPose Using Part Affinity Fields]]，通过添加一个新的用于生成RAF的卷积头来获取物体之间的关系。
![[Pasted image 20250317115153.png]]
具体RAF是如何定义的，以及如何训练(Loss)：
![[Pasted image 20250317115203.png]]
![[Pasted image 20250317115216.png]]


## Results
![[Pasted image 20250317121116.png]]
![[Pasted image 20250317121142.png]]
![[Pasted image 20250317121325.png]]

