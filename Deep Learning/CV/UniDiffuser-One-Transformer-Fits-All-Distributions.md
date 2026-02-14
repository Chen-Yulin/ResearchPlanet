---
id: UniDiffuser
tags:
  - DiffusionModel
  - Transformer
  - Multi-modal
  - ImgGen
  - Image2Text
  - CV
date:
  - 2026-02-03 12:30:00
categories: Review
---
# UniDiffuser: One Transformer Fits All Distributions in Multi-Modal Diffusion at Scale

[论文链接](https://arxiv.org/abs/2303.06555) | [GitHub](https://github.com/thu-ml/unidiffuser)

**作者**：Fan Bao, Shen Nie, Kaiwen Xue, Chongxuan Li, Shi Pu, Yaole Wang, Gang Yue, Yue Cao, Hang Su, Jun Zhu
**发表于**：ICML 2023

---
![[Pasted image 20260203153540.png]]

## 研究背景

扩散模型（Diffusion Models）在图像生成领域取得了巨大成功，但现有方法主要基于 U-Net 架构。随着 Transformer 在各领域的成功应用，如何将 Transformer 有效地应用于多模态扩散模型成为一个重要研究方向。现有的多模态生成方法通常需要为不同任务（文生图、图生文、联合生成等）设计不同的模型架构。

---

## 研究目标

- **多模态联合建模**：用一个统一的框架同时处理图像、文本等多种模态的生成任务
- **任务统一**：用单一模型支持文生图、图生文、联合生成、无条件生成等多种任务
- **架构创新**：设计适合多模态扩散的 Transformer 架构（U-ViT）

---

## 核心概念

### 扩散模型基础
扩散模型通过前向过程逐步向数据添加噪声，再通过反向过程学习去噪，从而实现生成。

### 多模态独立加噪
UniDiffuser 的核心思想是对不同模态**独立**添加噪声，使用不同的时间步 $t$ 和 $s$，通过控制时间步实现任务切换。

### U-ViT 架构
将 U-Net 的长跳跃连接（Long Skip Connection）引入 Vision Transformer，在浅层和深层之间建立连接。
![[Pasted image 20260203153607.png]]

---

## 研究方法

### 1. 单模态扩散基础公式

#### 前向过程（加噪）
$$q(x_t | x_0) = \mathcal{N}(x_t; \alpha_t x_0, \sigma_t^2 I)$$
**参数说明**：
- $x_0$：原始干净数据
- $x_t$：时间步 $t$ 的加噪数据
- $\alpha_t, \sigma_t$：噪声调度参数，满足 $\alpha_t^2 + \sigma_t^2 = 1$（VP-SDE 设定）
- 随着 $t$ 增大，$\alpha_t \to 0$，$\sigma_t \to 1$，数据逐渐变成纯噪声

**等价的重参数化表示**：
$$x_t = \alpha_t x_0 + \sigma_t \epsilon, \quad \epsilon \sim \mathcal{N}(0, I)$$
这个形式便于采样和训练，$\epsilon$ 是标准高斯噪声。

#### 反向过程（去噪）
扩散模型学习反向过程 $p_\theta(x_{t-1}|x_t)$，通过神经网络预测噪声 $\epsilon_\theta(x_t, t)$，然后恢复 $x_0$：
$$\hat{x}_0 = \frac{x_t - \sigma_t \epsilon_\theta(x_t, t)}{\alpha_t}$$
**解释**：由 $x_t = \alpha_t x_0 + \sigma_t \epsilon$ 反解得到。预测出噪声后，即可估计原始数据。

---

### 2. 多模态联合扩散框架

#### 联合前向过程
对于图文对 $(x_0, y_0)$，**独立**地对每个模态加噪：
$$q(x_t, y_s | x_0, y_0) = q(x_t | x_0) \cdot q(y_s | y_0)$$
**展开形式**：
$$x_t = \alpha_t x_0 + \sigma_t \epsilon_x, \quad y_s = \alpha_s y_0 + \sigma_s \epsilon_y$$
**关键点**：两个模态使用**独立的时间步** $t$ 和 $s$，这是实现多任务统一的核心设计。

#### 联合分布的边缘化
数据的联合分布 $q(x_0, y_0)$ 加噪后变为：
$$q(x_t, y_s) = \int q(x_t, y_s | x_0, y_0) q(x_0, y_0) \, dx_0 dy_0$$
**解释**：这是对所有可能的原始数据对进行积分，得到加噪后数据的边缘分布。

---

### 3. 统一多任务的核心机制

#### 核心洞察
通过控制 $t$ 和 $s$ 的取值，可以从联合分布中恢复各种边缘分布和条件分布：

| 时间步设置 | 对应分布 | 实现的任务 |
|------------|----------|------------|
| $t, s > 0$ | $q(x_t, y_s)$ | 联合生成 |
| $t > 0, s = 0$ | $q(x_t, y_0) = q(x_t \| y_0) q(y_0)$ | 文生图 |
| $t = 0, s > 0$ | $q(x_0, y_s) = q(y_s \| x_0) q(x_0)$ | 图生文 |
| $t > 0, s = T$ | $q(x_t, y_T) \approx q(x_t) q(y_T)$ | 无条件图像生成 |

#### 条件生成的数学原理
当 $s=0$ 时，$y_s = y_0$（文本无噪声），此时：
$$q(x_t, y_0) = q(x_t | y_0) \cdot q(y_0)$$
**解释**：对 $y$ 不加噪声（$s=0$）在数学上等价于以 $y$ 为条件进行生成。这是一个优雅的设计——不需要修改模型架构，只需控制时间步即可切换任务。

---

### 4. 训练目标函数

#### 噪声预测目标
模型 $\epsilon_\theta$ 同时预测两个模态的噪声：
$$[\hat{\epsilon}_x, \hat{\epsilon}_y] = \epsilon_\theta(x_t, y_s, t, s)$$
**完整训练损失**：
$$\mathcal{L}(\theta) = \mathbb{E}_{t, s, (x_0, y_0), \epsilon_x, \epsilon_y} \left[ \lambda_t \|\epsilon_x - \hat{\epsilon}_x\|^2 + \lambda_s \|\epsilon_y - \hat{\epsilon}_y\|^2 \right]$$
**各项说明**：
- $t, s \sim \mathcal{U}[0, T]$：从均匀分布采样时间步
- $(x_0, y_0) \sim q(x_0, y_0)$：从数据集采样图文对
- $\epsilon_x, \epsilon_y \sim \mathcal{N}(0, I)$：独立采样两个高斯噪声
- $\lambda_t, \lambda_s$：损失权重（通常设为 1）
- $\|\cdot\|^2$：均方误差损失

**简化形式**（实际训练中常用）：
$$\mathcal{L} = \mathbb{E}\left[\|\epsilon_x - \hat{\epsilon}_x\|^2 + \|\epsilon_y - \hat{\epsilon}_y\|^2\right]$$

---

### 5. 采样过程

#### 联合采样（DDPM 形式）
从 $t=T$（纯噪声）开始，逐步去噪到 $t=0$：
$$x_{t-1} = \frac{1}{\sqrt{\alpha_{t|t-1}}} \left( x_t - \frac{1-\alpha_{t|t-1}}{\sigma_t} \hat{\epsilon}_x \right) + \tilde{\sigma}_t z$$
$$y_{s-1} = \frac{1}{\sqrt{\alpha_{s|s-1}}} \left( y_s - \frac{1-\alpha_{s|s-1}}{\sigma_s} \hat{\epsilon}_y \right) + \tilde{\sigma}_s z'$$
**参数说明**：
- $z, z' \sim \mathcal{N}(0, I)$：采样的随机噪声（引入随机性）
- $\tilde{\sigma}_t$：后验方差，控制采样的随机程度
- $\alpha_{t|t-1} = \alpha_t / \alpha_{t-1}$：相邻时间步的比值

#### 条件采样（文生图）
固定 $s=0$（即 $y_s = y_0$ 为输入文本），只对图像进行去噪：
$$x_{t-1} = \frac{1}{\sqrt{\alpha_{t|t-1}}} \left( x_t - \frac{1-\alpha_{t|t-1}}{\sigma_t} \hat{\epsilon}_x(x_t, y_0, t, 0) \right) + \tilde{\sigma}_t z$$
**解释**：文本时间步固定为 0，模型接收干净文本作为条件，只更新图像。

---

### 6. U-ViT 架构中的跳跃连接

U-ViT 在第 $l$ 层和第 $(L-l)$ 层之间添加跳跃连接：
$$h^{(L-l)} = \text{Block}^{(L-l)}\left( \text{Concat}(h^{(L-l-1)}, h^{(l)}) \right)$$
**参数说明**：
- $h^{(l)}$：第 $l$ 层的隐藏状态
- $L$：Transformer 总层数
- Concat：沿特征维度拼接
- 拼接后通过线性层降维回原始维度

**设计动机**：借鉴 U-Net 的成功经验，跳跃连接帮助保留低层的细节信息，有助于生成高质量图像。

---

### 7. 分类器自由引导（Classifier-Free Guidance, CFG）

为增强条件生成效果，使用 CFG 技术：
$$\tilde{\epsilon}_x = \epsilon_\theta(x_t, \varnothing, t, 0) + w \cdot \left( \epsilon_\theta(x_t, y_0, t, 0) - \epsilon_\theta(x_t, \varnothing, t, 0) \right)$$
**参数说明**：
- $\varnothing$：空文本条件（null condition）
- $w$：引导强度（guidance scale），通常 $w > 1$
- 第一项：无条件预测
- 括号内：条件预测与无条件预测的差值（条件信号）

**等价形式**：
$$\tilde{\epsilon}_x = (1-w) \cdot \epsilon_\theta(x_t, \varnothing, t, 0) + w \cdot \epsilon_\theta(x_t, y_0, t, 0)$$

**训练技巧**：训练时以一定概率（如 10%）将文本随机替换为空文本，使模型同时学习条件和无条件生成。

---

## 模型输入总结

| 阶段 | 图像输入 | 文本输入 | 时间步 |
|------|----------|----------|--------|
| **训练** | 加噪图像 $x_t$ | 加噪文本 $y_s$ | $t, s$ 随机采样 |
| **联合生成** | 纯噪声 | 纯噪声 | $t=s=T \to 0$ |
| **文生图** | 纯噪声 | 原始文本 | $t: T \to 0$, $s=0$ |
| **图生文** | 原始图像 | 纯噪声 | $t=0$, $s: T \to 0$ |

---

## 主要发现

### 实验结果

**MS-COCO 256×256 文生图（零样本）**：

| 方法 | FID↓ | CLIP Score↑ |
|------|------|-------------|
| DALL-E | 27.50 | - |
| GLIDE | 12.24 | - |
| Stable Diffusion | 12.63 | 0.331 |
| **UniDiffuser** | **9.71** | **0.322** |

**ImageNet 256×256 类条件生成**：

| 方法 | FID↓ |
|------|------|
| ADM | 10.94 |
| LDM-4 | 10.56 |
| DiT-XL/2 | 9.62 |
| **U-ViT-H/2** | **2.29** |

---

## 讨论

### 优势
- **统一框架**：单一模型支持多种生成任务，无需为每个任务单独训练
- **优雅设计**：通过时间步控制实现任务切换，不需要修改架构
- **强大性能**：在多个基准上达到 SOTA
- **可扩展性**：在 10 亿参数规模上验证有效

### 局限性
- 需要大规模图文对数据进行训练
- 文本生成质量依赖于 CLIP 编码器的表示能力
- 推理速度受限于扩散模型的迭代采样

---

## 相关工作

- [DALL-E](https://arxiv.org/abs/2102.12092)：基于 VQ-VAE 的文生图模型
- [Stable Diffusion](https://arxiv.org/abs/2112.10752)：潜空间扩散模型
- [DiT](https://arxiv.org/abs/2212.09748)：Diffusion Transformer 架构[[Diffusion-Transformers-DiT]]
- [GLIDE](https://arxiv.org/abs/2112.10741)：文本引导的扩散模型

---

## 公式速查表

| 公式 | 含义 |
|------|------|
| $x_t = \alpha_t x_0 + \sigma_t \epsilon$ | 前向加噪过程 |
| $q(x_t, y_s \| x_0, y_0) = q(x_t\|x_0) q(y_s\|y_0)$ | 独立加噪（多任务统一的关键） |
| $\mathcal{L} = \|\epsilon_x - \hat{\epsilon}_x\|^2 + \|\epsilon_y - \hat{\epsilon}_y\|^2$ | 训练目标 |
| $s=0 \Rightarrow$ 以文本为条件 | 任务切换机制 |
| $\tilde{\epsilon} = \epsilon_\varnothing + w(\epsilon_y - \epsilon_\varnothing)$ | 分类器自由引导 |

---

## 参考文献

- Bao, F., et al. (2023). One Transformer Fits All Distributions in Multi-Modal Diffusion at Scale. ICML 2023.
- Ho, J., et al. (2020). Denoising Diffusion Probabilistic Models. NeurIPS 2020.
- Peebles, W., & Xie, S. (2023). Scalable Diffusion Models with Transformers. ICCV 2023.
