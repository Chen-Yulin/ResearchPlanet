---
id: (UVtransE) Contextual Translation Embedding for Visual Relationship Detection and Scene Graph Generation
tags:
  - Image-Text
  - Visual-Relation
  - Scene-graph
  - Translation-Embedding
date: 2025-03-18 16:04:44
categories: Note
---
![[Pasted image 20250318160643.png]]
The **Union Visual Translation Embedding network (UVTransE)**, which learns three projection matrices $W_{s}$, $W_{o}$, $W_{u}$ which map the respective feature vectors of the bounding boxes enclosing the subject, object, and union of subject and object into a common embedding space, as well as translation vectors $t_{p}$ (to be consistent with [[(VtransE) Visual Translation Embedding Network for Visual Relation Detection]]) in the same space corresponding to each of the predicate labels that are present in the dataset.