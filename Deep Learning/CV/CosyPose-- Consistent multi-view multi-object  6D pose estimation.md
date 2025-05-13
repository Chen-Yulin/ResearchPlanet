---
id: CosyPose-- Consistent multi-view multi-object  6D pose estimation
tags:
  - CV
  - PoseEstimation
  - Multi-view
date: 2024-11-07 17:28:05
categories: Review
---
## Goal
Estimate accurate 6D poses of multiple **known** objects in a 3D scene captured by **multiple cameras with unknown positions**

![[Pasted image 20241112222444.png]]

### Challenges
- object pose hypotheses made in individual images cannot easily be expressed in a common reference frame when the relative transformations between the cameras are unknown(相机相对位置未知)
- the single-view 6D object pose hypotheses have gross errors in the form of false positive and missed detections（由于视角遮蔽，会存在误报和错漏的情况）
- the candidate 6D object poses estimated from input images are noisy as they suffer from depth ambiguities inherent to single view methods.（深度信息通常没那么精准）

## Approach
![[Pasted image 20241112224221.png]]
