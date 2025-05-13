---
id: Augmented Reality and Robotics - A Survey and Taxonomy for AR-enhanced Human-Robot Interaction and Robotic Interfaces
tags:
  - HRI
  - MR/AR
  - Robotics
  - Survey
date: 2024-10-28 15:48:36
categories: Review
---
## 概要
虽然近些年有关AR在人机交互方面应用的研究有很多，但是这些研究大都缺少**系统性的分析**
> Recently, an increasing number of studies in HCI, HRI, and robotics have demonstrated how AR enables better interactions between people and robots. However, often research remains focused on individual explorations and key design strategies, and research questions are rarely analyzed systematically.

本文主要给目前AR人机交互领域做一下分类（基于460篇文章）
AR人机交互主要分为这几种研究维度
- approaches to augmenting reality
- characteristics of robots
- purposes and benefits
- classification of presented information
- design components and strategies for visual augmentation
- interaction techniques and modalities
- application domains
- evaluation strategies

![[Pasted image 20241029135747.png]]


AR最大的优势就是能够提供超出物理限制的丰富视觉反馈，减少工人的认知负荷
这个研究最终的目标是提供一个对于该领域的共同基础和理解。

## Definition, Scope, Contribution, Methodology

### HRI & Robotic Interfaces
机器人系统不单指传统工业机器人，在本研究中，我们不局限于任一种机器人。
Robotic interfaces 主要指"Interfaces that use robots or other actuated systems as medium for HCI".

### Contribution
该研究通过design space dimensions来呈现该领域的分类
拓宽了HCI和HRI的文献研究
讨论了促进该领域进一步研究的开放性研究问题和机会
有一个交互式网站 https://ilab.ucalgary.ca/ar-and-robotics/

![[Pasted image 20241029142609.png]]

## 分类

### Approaches to augmenting reality
根据增强现实硬件的布置位置（dimension 1），可以分为
- on-body
- on-environment
- on-robot
根据视觉增强的目标位置（dimension 2），可以分为
- augmenting robots
- augmenting surroundings

![[Pasted image 20241103132127.png]]


### Characteristics of robots
1) the form factor of robots （机器人类型）
2) the relationship between the users and robots （n:m）
3) size and scale of the robots
4) proximity for interactions (交互距离)

![[Pasted image 20241103135953.png]]

### Purposes and benefits
1) Facilitate Programming （类似毕设）
   - 在虚拟3D空间中编辑，可视化路径
   - 通过物体识别把现实物体映射到虚拟空间用于抓取
1) Support Real-time Control and Navigation
2) Improve Safety （类似毕设中的碰撞检测急停）
3) Communicate Intent （绘制机器人的意向轨迹）
4) Increase the Expressiveness （机器人的虚拟义体）

![[Pasted image 20241103141336.png]]


### Classification of presented information
1) robot’s internal information
	1) robot’s internal status
	2) robot’s software and hardware condition
	3) robot’s internal functionality and capability
2) external information about the environment
	1) sensor data from the internal or external sensors
	2) camera or video feed
	3) information about external objects
	4) depth map or 3D reconstructed scene of the environment (就是hololens的环境感知网格)
3) plan and activity
	1) a plan of the robot’s motion and behavior
	2) simulation results of the programmed behavior
	3) visualization of a target and goal
	4) progress of the current task
4) supplemental content

![[Pasted image 20241104113403.png]]

### Design components and strategies for visual augmentation
这篇主要讨论呈现AR内容的方式
1) UIs and Widgets
	1) Menus
	2) Information Panels
	3) Labels and Annotations
	4) Controls and Handles
	5) Monitors and Displays
2) Spatial References and Visualizations (将空间3D图像叠加显示到现实空间中)
	1) Points and Locations
	2) Paths and Trajectories
	3) Areas and Boundaries
	4) Other Visualizations（比如空间颜色/热图可视化）
3) Embedded Visual Effects （相较于Spatial References and Visualizations，不需要包含数据信息）
	1) anthropomorphic effects
	2) virtual replica
	3) texture mapping of physical objects
4) Anthropomorphic Effects (机器人的社交拟人内容（谁来给增强一下社交拟人功能）)
5) Virtual Replica and Ghost Effects （虚拟物品）
6) Texture Mapping Effects based on Shape (例如给衣服换个图案)

![[Pasted image 20241104114741.png]]

### Interactions
1) Dimension-1. Level of Interactivity
![[Pasted image 20241104161433.png]]
2) Dimension-2. Interaction Modalities
![[Pasted image 20241104161449.png]]

### Application Domains
![[Pasted image 20241104163809.png]]

### Evaluation strategies
1) Evaluation through Demonstration (诸如在Seminar,workshop展示功能，示例程序)
2) Technical Evaluation
	1) 延迟测量
	2) 物体跟踪误差
	3) 成功率
	4) 与其他系统的对比（tracking system for example）
3) User Evaluation（通过访谈，问卷，通常和前两者结合）

## Future
1) 使AR-HR更具实用性
	1) 头戴式AR设备的追踪误差（陀螺仪），可靠性仍需加强
	2) 在户外使用的局限性
2) 对AR HRI的新的设计探索
	1) 可以依靠AR设计不局限于物理限制的机器人
	2) 更好的开发环境（因为目前的AR开发仍然主要使用平面显示器，可以思考有没有基于AR显示做程序设计的应用）
3) AR for better decision making(针对用户)
	1) 可视化场景数据
	2) 可解释性的机器人操作
4) 新颖交互设计
	1) 更自然的交互方式（例如更自然地指定任务对象）
	2) 进一步融合虚拟和物理世界（让虚拟的交互能影响现实物理（经典最扯的放最后））

