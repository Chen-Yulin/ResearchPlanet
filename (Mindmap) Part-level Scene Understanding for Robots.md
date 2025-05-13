---
id: (Mindmap) Part-level Scene Understanding for Robots
tags:
  - Robotics
  - Scene-graph
  - Visual-Relation
date: 2025-04-14 12:12:22
categories: Note
---
## 概念梳理
### Scene Graph
A scene graph is a structural representation, which can capture detailed semantics by explicitly Modeling:
- objects (‘‘man’’, ‘‘fire hydrant’’, ‘‘shorts’’)
- attributes of objects (‘‘fire hydrant is yellow’’)
- relations between paired objects (‘‘man jumping over fire hydrant’’)

A scene graph is a set of **visual relationship** triplets in the form of <subject, relation, object> or <object, is, attribute>
![[Pasted image 20250414142333.png]]
Scene graphs should serve as an **objective semantic representation** of the state of the scene

