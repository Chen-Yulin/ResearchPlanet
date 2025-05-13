---
id: Momentum Contrast for Unsupervised Visual Representation Learning
tags:
  - Contrastive-Learning
date: 2025-01-09 19:48:43
categories: Note
aliases: 
publish: "2020"
---
![[Pasted image 20250304154105.png]]
左侧是query encoder，右侧为key encoder
## 是什么
通过无监督对比学习的方法(loss:InfoNCE)来学习图像的特征。
![[Pasted image 20250304154327.png]]
使用的pretext task是个体判别任务

伪代码：
```python
# f_q, f_k: encoder networks for query and key 
# queue: dictionary as a queue of K keys (CxK) 
# m: momentum 
# t: temperature  

f_k.params = f_q.params # initialize 
for x in loader: # load a minibatch x with N samples 
	x_q = aug(x) # a randomly augmented version 
	x_k = aug(x) # another randomly augmented version  
	q = f_q.forward(x_q) # queries: NxC 
	k = f_k.forward(x_k) # keys: NxC 
	k = k.detach() # no gradient to keys  
	
	# positive logits: Nx1 
	l_pos = bmm(q.view(N,1,C), k.view(N,C,1))  # 相当于把batch中每个正样本对之间求了cosine临近
	
	# negative logits: NxK 
	l_neg = mm(q.view(N,C), queue.view(C,K))  
	
	# logits: Nx(1+K) 
	logits = cat([l_pos, l_neg], dim=1)  
	
	# contrastive loss, Eqn.(1) 
		labels = zeros(N) # positives are the 0-th，将识别的类别视为0,可以直接使用CrossEntropyLoss
	loss = CrossEntropyLoss(logits/t, labels)  
	
	# SGD update: query network 
	loss.backward() 
	update(f_q.params)  
	
	# momentum update: key network 
	f_k.params = m*f_k.params+(1-m)*f_q.params  
	
	# update dictionary 
	enqueue(queue, k) # enqueue the current minibatch 
	dequeue(queue) # dequeue the earliest minibatch

```

## 亮点

### Dictionary as a queue
在使用key encoder(momentum encoder)创建负样本，并把encode过的负样本存在一个queue（FIFO）中方便后续对比时直接使用，每次训练都会使用一个新的mini batch，此时会将此mini batch中的样本encode之后加入queue并删除存在最久的那个mini batch的样本（因为考虑到最老的mini batch使用的encoder是最过时的，所以FIFO是非常合理的），这样可以有效控制负样本的数量，也就是公式中的K。
- 节省字典的计算开销
- 而且mini batch大小可以直接和负样本脱钩

### Momentum update
因为负样本数量（字典/队列）很大，所以没办法给key encoder回传梯度，所以可以考虑把query encoder的参数直接复制给key encoder，**但过快改变的key encoder会导致样本字典的特征不一致**，所以使用动量更新的方式。
![[Pasted image 20250304160138.png]]
> queue这个字典越大，那么理论上这个m就需要越大，保证字典中key的一致性

## 过往工作对比
![[Pasted image 20250304160703.png]]
a)
所有的样本都在一个 mini batch 里，两个encoder完全一致，也因此都可以回传梯度，keys也高度一致，但限制了字典的大小

b)
只有一个编码器进行学习。Memory bank存下了所有样本的key。每当梯度回传后，会把memory bank被本次训练中被采样过的key使用新的encoder进行更新。
- 缺乏特帧一致性
- 需要训练一阵个epoch才能更新一遍memory bank

MoCo和memory bank 更接近，但是使用了queue dictionary和momentum update

