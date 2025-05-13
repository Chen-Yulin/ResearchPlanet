---
id: SGTR+= End-to-end Scene Graph Generation with Transformer
tags:
  - Visual-Relation
  - Transformer
  - Scene-graph
date: 2025-03-13 12:05:10
categories: Review
publish: "2022"
---
![[Pasted image 20250314152838.png]]

SGTR 是一种自上而下的方法，该方法首先使用基于Transformer的生成器来生成一组可学习的triplet queries (subject–predicate–object)，然后使用级联的triplet detector逐步完善这些查询并生成最终场景图。它还提出了一种基于结构化发生器的实体感知关系表示方法，该方法利用了关系的组成属性。

> **Top-down approach (SGTR):**
	- Starts with higher-level structures (triplet queries) and refines them
	- Begins by generating complete subject-predicate-object triplet candidates
	- Then progressively refines these triplets to match the image content
	- Works with the complete structural units from the beginning
	- Analogous to starting with a rough sketch of the entire tree and then refining each branch

![[Pasted image 20250315170430.png]]
![[Pasted image 20250315170407.png]]
![[Pasted image 20250315170416.png]]