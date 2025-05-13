---
id: DINO
tags:
  - DINO
  - ViT
  - Transformer
  - CV
date: 2025-01-08 20:26:27
categories: Note
publish: "2021"
---
https://github.com/facebookresearch/dino/tree/main

![[Pasted image 20250306112727.png]]
# Emerging Properties in Self-Supervised Vision Transformers

https://juejin.cn/post/7224738994825789496
https://www.youtube.com/watch?v=h3ij3F3cPIk&t=1005s
DI+NO（蒸馏+No Label）
具体来说，DINO 是使用一种称为“无监督自蒸馏”的方法，该方法通过自监督学习来学习模型的知识表示。在这个方法中，模型使用自身的输出来生成“伪标签”，然后使用这些伪标签来重新训练模型，从而进一步提高模型的性能和泛化能力。

## 知识蒸馏
https://blog.csdn.net/xbinworld/article/details/83063726
> 重点idea就是提出用soft target来辅助hard target一起训练，而soft target来自于大模型的预测输出。这里有人会问，明明true label（hard target）是完全正确的，为什么还要soft target呢？
hard target 包含的信息量（信息熵）很低，soft target包含的信息量大，拥有不同类之间关系的信息（比如同时分类驴和马的时候，尽管某张图片是马，但是soft target就不会像hard target 那样只有马的index处的值为1，其余为0，而是在驴的部分也会有概率。）[5]
这样的好处是，这个图像可能更像驴，而不会去像汽车或者狗之类的，而这样的soft信息存在于概率中，以及label之间的高低相似性都存在于soft target中。但是如果soft targe是像这样的信息[0.98 0.01 0.01]，就意义不大了，所以需要在softmax中增加温度参数T（这个设置在最终训练完之后的推理中是不需要的）
![[Pasted image 20250108203323.png]]

##  ViT
![[Pasted image 20250109141410.png]]


## DINO 
![[Pasted image 20250109152153.png]]
![[Pasted image 20250304154624.png]]
总的来说DINO最适合的任务就是将不同状态的同一物体进行归类。


关于DINO中发生的涌现
https://juejin.cn/post/7280436457142501388

DINO之前的工作
![[Pasted image 20250110172355.png]]

We have also seen emerged two properties that can be leveraged in future applications: the quality of the features in k-NN classification has a potential for image retrieval. The presence of information about the scene layout in the features can also benefit weakly supervised image segmentation.
![[Pasted image 20250219093931.png]]