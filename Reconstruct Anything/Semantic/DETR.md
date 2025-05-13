---
id: DETR
tags:
  - Object-Detection
  - Transformer
date: 2025-03-11 16:15:11
categories: Review
publish: "2020"
---
![[Pasted image 20250311161649.png]]
![[Pasted image 20250311163349.png]]![[Pasted image 20250311163752.png]]
参考： https://www.youtube.com/watch?v=T35ba_VXkMY&t=1744s

DETR是一个使用transformer作为基本架构的 object detection 模型。

## Insight
Object queries (something that can be learned):
![[Pasted image 20250311164555.png]]
>Visualization of all box predictions on all images from COCO 2017 val set for 20 out of total N = 100 prediction slots in DETR decoder. Each box prediction is represented as a point with the coordinates of its center in the 1-by-1 square normalized by each image size. The points are color-coded so that green color corresponds to small boxes, red to large horizontal boxes and blue to large vertical boxes. We observe that each slot learns to specialize on certain areas and box sizes with several operating modes. We note that almost all slots have a mode of predicting large image-wide boxes that are common in COCO dataset.


