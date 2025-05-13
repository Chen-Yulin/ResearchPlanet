---
id: ACDC- Automated Creation of Digital Cousins for Robust Policy Learning
tags:
  - Reconstruct
  - CV
  - LLM
  - Robotics
  - Real2Sim
  - DT
  - Sim2Real
date: 2024-12-09 15:40:40
categories: Review
publish: 2024 Oct
---
![[Pasted image 20241209154449.png]]

## Intro
数字孪生(DT)作为现实世界非常精确的映射虽然可以用于高精度的训练但是生产DT资产过于繁琐且没有泛化性，不能做到zero-shot。
数字表亲(DC)通过比对模型特征，从模型库中选择类似的表亲模型，用于重建场景训练机械臂。让机械臂针对不同第一次见的场景具有**泛化性**。
（a）它减少了手动微调的需要，以保证一定的保真度，从而能够**完全自动化地创建数字表亲**，（b）它通过提供一组增强的场景来训练机器人策略，从而有助于更好地应对原始场景中的变化。

## Methodology
ACDC is our automated pipeline for generating fully interactive simulated scenes from a single RGB image, and is broken down into three steps:
	(1) an extraction step, in which relevant object masks are extracted from the raw input image
	(2) a matching step, in which we select digital cousins for individual objects extracted from the original scene
	(3) a generation step, in which the selected digital cousins are post-processed and compiled together to form a **fully-interactive, physically-plausible** digital cousin scene.



