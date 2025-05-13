---
id: Associative Embedding= End-to-End Learning for Joint Detection and Grouping
tags:
  - CV
  - PoseEstimation
  - Object-Detection
date: 2025-04-24 11:00:20
categories: Review
publish:
---
## Q&A
### 1
What is standard dense supervised learning? Mentioned in [[CenterNet]].

**Standard dense supervised learning** typically refers to a supervised learning setup where:
1. **Standard supervised learning** means:
    - You have input data X and corresponding ground truth labels Y.
    - The goal is to train a model $f_\theta(X)$ that maps inputs to outputs by minimizing a loss function (e.g., cross-entropy, MSE) between the predicted labels and ground truth.
    - The training dataset is fully labeled (i.e., each input has a corresponding label).
2. **Dense** refers to:
    - A **per-pixel** or **per-element** prediction task, where every element in the input gets a corresponding label.
    - Common in vision tasks like:
        - **Semantic segmentation** (each pixel is labeled with a class).
        - **Depth estimation** (each pixel has a depth value).
        - **Optical flow** (each pixel has a motion vector).
        - **Surface normal estimation** (each pixel has a 3D orientation vector).

In contrast to **sparse supervision**, where only a subset of the input (e.g., bounding boxes, keypoints) is labeled, **dense supervision** provides full annotations for every relevant part of the input.

**Example**
In semantic segmentation:
- Input: an RGB image (e.g., 512×512 pixels).
- Output: a label map of the same size (512×512), where each pixel has a class label like "road", "car", "sky", etc.
- Model: often a Fully Convolutional Network (FCN) or encoder-decoder like U-Net or DeepLab.
- Loss: usually pixel-wise cross-entropy.