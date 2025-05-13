---
id: Grounding-DINO
tags:
  - CV
  - DINO
  - Open-Vocabulary
  - Object-Detection
  - Image-Grounding
  - Multi-modal
date: 2025-02-16 17:07:36
categories: Review
publish: 2024 April
---
,![[Pasted image 20250219103227.png]]

通过结合[[DINO]]和grounded-pretraining，可以使用人类输入（例如类别名称或转介表达式）检测任意对象
Open-Vocab. Det
> an open-set object detector that can detect any objects with respect to an arbitrary free-form text prompt. The model was trained on over 10 million images, including detection data, visual grounding data, and image-text pairs. It has a strong zero-shot detection performance. However, the model needs text as inputs and can only detect boxes with corresponding phrases.
## Grounding-DINO
### Principle
#### Tight modality fusion based on [[DINO]]
什么是feature fusion?
![[Pasted image 20250219100532.png]]
- 在多模态领域，feature fusion 特指将不同模态的特征（如视觉、文本、音频等）进行融合的技术。CLIP 应该被看作是 Middle Fusion 的一种形式, 在特征提取后就进行融合对齐
#### large-scale grounded pre-train for concept generalization
Reformulating **object detection** as a **phrase grounding task** and introducing **contrastive training** between object regions and language phrases on large-scale data