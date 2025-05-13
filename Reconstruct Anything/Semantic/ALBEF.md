---
id: ALBEF
tags:
  - Multi-modal
  - Image-Text
date: 2025-03-04 12:31:04
categories: Review
publish:
---
![[Pasted image 20250304123316.png]]
## Align Before Fuse
### Image Encoder
标准的六层self attention ViT，初始化为DeiT论文中的在ImageNet-1K数据集上训练的参数

### Text Encoder
使用的backbone是BERT(通过MLM训练)
该研究认为，image encoder的模型大小应该大于text encoder,所以在text encoder这里，只使用六层self attention来提取特征，剩余六层cross attention用于multi-modal encoder。

### ITC Loss & Momentum
参考Moco [[Moco- Momentum Contrast for Unsupervised Visual Representation Learning]]

## Improve Noisy Web Data
见[[BLIP]]，是沿用的工作

### Loss
![[Pasted image 20250304190649.png]]
#### ITC
旨在在融合之前学习更好的单模态表示

#### ITM

#### MLM





