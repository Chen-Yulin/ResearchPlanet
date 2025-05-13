---
id: CLIP
tags:
  - CV
  - Multi-modal
  - Image-Text
  - Contrastive-Learning
date: 2025-01-06 20:12:44
categories: Note
---
![[Pasted image 20250106201326.png]]

https://blog.csdn.net/h661975/article/details/135116957

loss: ITC (Image Text Contrastive)
```python
# image_encoder - ResNet or Vision Transformer 
# text_encoder - CBOW or Text Transformer 
# I[n, h, w, c] - minibatch of aligned images 
# T[n, l] - minibatch of aligned texts 
# W_i[d_i, d_e] - learned proj of image to embed 
# W_t[d_t, d_e] - learned proj of text to embed 
# t - learned temperature parameter  

# extract feature representations of each modality 
I_f = image_encoder(I) #[n, d_i] 
T_f = text_encoder(T) #[n, d_t]  

# joint multimodal embedding [n, d_e] 
I_e = l2_normalize(np.dot(I_f, W_i), axis=1) T
_e = l2_normalize(np.dot(T_f, W_t), axis=1)  

# scaled pairwise cosine similarities [n, n] 
logits = np.dot(I_e, T_e.T) * np.exp(t)  

# symmetric loss function 
labels = np.arange(n) 
loss_i = cross_entropy_loss(logits, labels, axis=0) 
loss_t = cross_entropy_loss(logits, labels, axis=1) 
loss = (loss_i + loss_t)/2
```

Cross_entropy_loss:
![[Pasted image 20250304121717.png]]

CLIP 本质上是全局图像嵌入，不利于像素对齐特征提取。