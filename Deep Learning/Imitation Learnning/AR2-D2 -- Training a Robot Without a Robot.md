---
id: AR2-D2 -- Training a Robot Without a Robot
tags:
  - ImitationLearning
  - MR/AR
  - Robotics
date: 2024-11-05 18:06:46
categories: Review
---
https://ar2d2.site/

## 背景
机器人执行任务的视频数据集非常重要，特别是对于Visual Imitation Learning来说。
想要获得这些训练集视频，传统的方法是人工引导机器人做相关动作，然后再录制，耗费大量人力和时间成本，最关键的是机器人是固定在实验室内的，能接触到的物品和任务比较有限，因此这些训练数据中不包含更日常的场景。

![[Pasted image 20241106095202.png]]


## Solution
提出了一个IOS APP，可以通过追踪用户手部的动作在视频中生成一个执行动作的AR机器人。

![[Pasted image 20241106114807.png]]

### AR2-D2 系统细节

![[Pasted image 20241106095410.png]]

如上图，AR2-D2 的设计和实现由两个主要组件组成。第一个组件是一个手机应用程序，它将 AR 机器人投射到现实世界中，允许用户与物理对象和 AR 机器人进行交互。第二个组件将收集的视频转换为可用于训练不同行为克隆代理的格式，这些克隆代理随后可以部署在真实的机器人上。
### IOS Application
Unity + AR Foundation kit（用于生成一个虚拟机械臂并布置在场景中）
传感器：苹果设备摄像头和自带的LiDAR
通过ios自己的人手姿态算法和深度信息获取手部动作，由此获取机械臂需要运动到的关键点，并且可以让AR界面中的机械臂移动到指定位置。

### Training Data Generation
得到APP生成的视频后消除人手并填补消除的区域（E2FGVI），就可以得到机械臂操作物体的视频，它可以用作基于视觉的模仿学习的训练数据。

## APP Evaluation

![[Pasted image 20241106121849.png]]

## Real Deployment Evaluation
围绕三个常见的机器人任务收集演示：{press, push, pick up}

使用 Perciver-Actor (PERACT)训练基于 Transformer 的语言引导行为cloning policy
> PERACT takes a 3D voxel observation and a language goal (v, l) as input and produces **discretized outputs** for translation, rotation, and gripper state of the end-effector. These outputs, coupled with a motion planner, enable the execution of the task specified by the language goal.

每一个agent执行一种任务（{press, push, pick up}），先训练3k次，然后再微调训练（3k iteration），用于缩小iphone摄像机和agent使用的kinect v2相机之间的偏差。

微调结果
![[Pasted image 20241106133117.png]]

测试结果
![[Pasted image 20241106133421.png]]
