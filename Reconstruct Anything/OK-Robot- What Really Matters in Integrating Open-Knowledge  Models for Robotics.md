---
id: OK-Robot- What Really Matters in Integrating Open-Knowledge  Models for Robotics
tags:
  - Robotics
  - CV
  - Semantic
  - Reconstruct
  - LLM
  - Open-Vocabulary
  - Embodied-AI
date: 2025-01-06 13:59:55
categories: Note
publish: 2024 Feb
---
![[Pasted image 20250106151641.png]]

## Intro
*Creating a general-purpose robot has been a longstanding dream of the robotics community.*

## 背景
当前想要实现这一目标的系统脆弱、封闭，并且在遇到未见过的情况时会失败。即使是最大的机器人模型通常也只能部署在以前见过的环境中 [5, 6]。在机器人数据很少的环境中，例如在非结构化的家庭环境中，这些系统的脆弱性会进一步加剧。

虽然大型视觉模型显示出语义理解 、检测以及将视觉表示与语言联系起来的能力并且与此同时，机器人的导航、抓取和重新排列等基本机器人技能已经相当成熟。
但是将现代视觉模型与机器人特定基元相结合的机器人系统表现非常差。

这可能是因为单纯将多个不确定性的系统组合在一起会导致准确率急剧恶化。
所以我们需要一个将VLM和机器人primitives(导航，抓取，放置)结合在一起的细致框架，即OK-Robot。

## 发现
- **预训练的 VLM 对于开放词汇导航非常有效**: 当前的开放词汇视觉语言模型，例如 CLIP 或 OWL-ViT，在识别现实世界中的任意对象方面提供了强大的性能，并能够以零样本的方式导航到它们。
- **预训练的抓取模型可以直接应用于移动操作**：与 VLM 类似，经过大量数据预训练的专用机器人模型可以立即应用于家庭中的开放词汇抓取。这些机器人模型不需要任何额外的训练或微调。
- **如何组合组件至关重要**：给定预训练模型，我们发现可以使用简单的状态机模型将它们组合在一起，无需训练。我们还发现，使用启发式方法来抵消机器人的物理限制可以在现实世界中获得更高的成功率。
- **仍然存在一些挑战**：虽然，考虑到在任意家庭中进行零样本的巨大挑战，OK-Robot 在之前的工作基础上进行了改进，通过分析故障模式，我们发现 VLM、机器人模型和机器人形态可以进行重大改进，这将直接提高开放知识操纵代理的性能。

##  Methodology
### 该框架主要完成的任务
Pick up A (from B) and drop it on/in C”, where A is an object and B and C are places in a real-world environment such as homes

### Open-home, open-vocabulary object navigation
负责空间重建，识别物体大致位置，机器人导航
用到的方法:
- CLIP-Fields [[CLIP-Fields- Weakly Supervised Semantic Fields for Robotic Memory]] : a RGB-D video of the home -> a sequence of posed ( with camera pose and positions) RGB-D images，用于重建环境，该研究还基于此获取了环境中物体和容器旁边的地板表面。
- OWL-ViT [[Simple Open-Vocabulary Object Detection with Vision Transformers]] : 我们在每一帧上应用检测器，并提取每个对象边界框、CLIP-embedding、检测器置信度，并将这些信息传递到object memory模块中
- SAM: 用于将ViT的检测框转化为mask
- VoxcelMap: similar to object-centric memory of CLIP-Fields [[CLIP-Fields- Weakly Supervised Semantic Fields for Robotic Memory]], 基于点云中每一个点的CLIP semantic vector,每一个5cm的体素都包含一个CLIP-embedding的detector-confidence weighted average.
- Querying the memory module: 先将language query 转化成CLIP semantic vector,然后基于voxelmap的clip-embeding，寻找最语义接近的那个voxel，以此定位。
- ![[Pasted image 20250106151051.png]]

## Experiment

![[Pasted image 20250106160701.png]]








