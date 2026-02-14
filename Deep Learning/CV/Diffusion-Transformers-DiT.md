---
id: Scalable Diffusion Models with Transformers
tags:
  - DiffusionModel
  - Transformer
  - ImgGen
  - Scalability
  - CV
date:
  - 2026-02-03 11:30:00
categories: Review
---
![[Pasted image 20260203120620.png]]
# Diffusion Transformers (DiT)

[Scalable Diffusion Models with Transformers](https://arxiv.org/abs/2212.09748) | ICCV 2023

---

## 研究背景

扩散模型(Diffusion Models)在图像生成领域取得了显著成功，但其架构设计一直沿用卷积U-Net作为主干网络。与此同时，Transformer架构已经在自然语言处理、视觉识别等多个领域取得了统治地位，并展现出优秀的可扩展性。本文探索将Transformer架构引入扩散模型，研究其在图像生成任务中的可扩展性和性能表现。

---

## 研究目标

- **突破架构局限**：探索用Transformer替代传统U-Net作为扩散模型主干的可行性
- **验证可扩展性**：研究Transformer在扩散模型中的可扩展性规律
- **建立性能基准**：在ImageNet等基准数据集上达到SOTA性能
- **揭示计算-质量关系**：分析模型计算量(Gflops)与生成质量之间的关系

---

## 核心概念

### Latent Diffusion Models (LDMs)

在潜在空间而非像素空间训练扩散模型，提高计算效率：

1. **VAE编码器**：将图像压缩到潜在空间 $z = E(x)$
2. **扩散模型**：在潜在空间 $z$ 中训练
3. **VAE解码器**：将生成的潜在表示解码为图像 $x = D(z)$

对于256×256×3的图像，VAE将其压缩为32×32×4的潜在表示（下采样因子为8）。

**注意**：这里使用的是标准VAE，输出是**连续的潜在表示**，而非VQ-VAE的离散codebook索引。

### Patchify机制

将潜在表示分解为patch序列：

- 输入：32×32×4的潜在表示
- Patch大小：$p \times p$（$p \in \{2, 4, 8\}$）
- 输出序列长度：$T = (I/p)^2$

例如，$p=2$ 时，序列长度 $T = (32/2)^2 = 256$。

### Classifier-Free Guidance

条件生成的采样技巧，提高生成质量：

$$
\hat{\epsilon}_\theta(x_t, c) = \epsilon_\theta(x_t, \emptyset) + s \cdot (\epsilon_\theta(x_t, c) - \epsilon_\theta(x_t, \emptyset))
$$

其中：
- $c$：条件信息（如类别标签）
- $\emptyset$：空条件（训练时随机dropout）
- $s$：guidance scale（$s > 1$增强条件控制）

---

## 研究方法

### DiT架构设计

DiT基于Vision Transformer (ViT)架构，包含以下组件：

1. **Patchify层**：将潜在表示转换为token序列
2. **位置编码**：使用正弦-余弦位置编码
3. **DiT Blocks**：N个Transformer block
4. **解码器**：将token序列解码为噪声预测和协方差预测

### 条件注入机制

探索了四种将时间步 $t$ 和类别标签 $c$ 注入Transformer的方式：

#### 1. In-Context Conditioning
将 $t$ 和 $c$ 的embedding作为额外token添加到序列中：

```python
tokens = [t_embed, c_embed, patch_1, patch_2, ..., patch_T]
```

- **优点**：无需修改标准Transformer block
- **缺点**：性能较差（FID ~80）

#### 2. Cross-Attention
通过交叉注意力机制注入条件：

```python
# 标准self-attention后添加cross-attention
x = x + SelfAttention(x)
x = x + CrossAttention(x, [t_embed, c_embed])
```

- **优点**：灵活的条件控制
- **缺点**：增加15%计算量，性能中等（FID ~60）

#### 3. Adaptive Layer Norm (adaLN)
通过自适应归一化层注入条件：

```python
γ, β = MLP(t_embed + c_embed)
output = γ * normalize(x) + β
```

- **优点**：计算高效，性能较好（FID ~45）
- **缺点**：所有token共享���同的调制参数

#### 4. adaLN-Zero（最优方案）
在adaLN基础上增加门控参数并零初始化：

```python
γ, β, α = MLP(t_embed + c_embed)

# Transformer Block
x = x + α₁ * Attention(γ₁ * normalize(x) + β₁)
x = x + α₂ * FFN(γ₂ * normalize(x) + β₂)

# 初始化：MLP输出零向量 → α=0, γ=0, β=0
# 因此初始时：x = x + 0 = x（恒等函数）
```

**为什么需要两组参数？**

Transformer block有两个子层（Attention + FFN），每个子层需要独立的条件控制：
- 第一组 $(γ_1, β_1, α_1)$：用于Attention子层
- 第二组 $(γ_2, β_2, α_2)$：用于FFN子层

总共6个参数：`shift_msa, scale_msa, gate_msa, shift_mlp, scale_mlp, gate_mlp`

**零初始化的优势**：
- 训练稳定性：初始时网络是恒等映射，梯度流动顺畅
- 更好的性能：FID显著优于其他方法（FID ~23）
- 渐进式学习：从恒等函数开始，逐步学习有用的变换

### 模型配置

设计四种规模的模型配置：

| 模型 | 层数N | 隐藏维度d | 注意力头数 | Gflops (p=4) |
|------|-------|-----------|-----------|--------------|
| DiT-S | 12 | 384 | 6 | 1.4 |
| DiT-B | 12 | 768 | 12 | 5.6 |
| DiT-L | 24 | 1024 | 16 | 19.7 |
| DiT-XL | 28 | 1152 | 16 | 29.1 |

### Point-wise FFN

Transformer中的标准组件，对序列中每个位置独立应用前馈网络：

```python
class PointwiseFeedForward(nn.Module):
    def forward(self, x):
        # x: [batch, seq_len, d_model]
        x = Linear1(x)      # d_model → d_ff (通常4倍扩展)
        x = GELU(x)
        x = Linear2(x)      # d_ff → d_model
        return x

# 关键特点：
# 1. 每个token独立处理（point-wise）
# 2. 所有位置共享相同的权重
# 3. 可以完全并行计算
```

**与Self-Attention的分工**：
- Self-Attention：全局信息聚合（不同token间交互）
- Point-wise FFN：局部特征变换（每个token独立处理）

这是**所有Transformer变体**（LLM、ViT、DiT）的统一设计。

---

## 主要发现

### 1. Gflops与生成质量强相关

模型前向传播的计算量(Gflops)与FID呈强负相关（相关系数-0.93）：

- 增加模型深度/宽度 → 提升Gflops → 降低FID
- 减小patch大小 → 增加token数量 → 提升Gflops → 降低FID

**关键洞察**：参数量不是���一决定因素，计算量才是提升性能的关键。

### 2. adaLN-Zero显著优于其他条件注入方式

在400K训练步数时的FID对比：

| 方法 | FID-50K | 计算开销 |
|------|---------|----------|
| In-Context | ~80 | 119.4 Gflops |
| Cross-Attention | ~60 | 137.6 Gflops |
| adaLN | ~45 | 118.6 Gflops |
| **adaLN-Zero** | **~23** | **118.6 Gflops** |

### 3. 优秀的可扩展性

DiT展现出与ViT类似的可扩展性：
- 增加模型规模持续提升性能
- 训练高度稳定，无需学习率预热或特殊正则化
- 未观察到常见的loss spike现象

### 4. 计算效率优势

DiT-XL/2 (118.6 Gflops) 比传统方法更高效：
- 像素空间U-Net (ADM)：1120 Gflops（~10倍）
- 潜在空间U-Net (LDM-4)：103.6 Gflops（相近但性能更优）

### 5. 采样计算无法弥补模型计算不足

增加采样步数（增加测试时计算量）无法弥补模型规模不足：
- DiT-L/2 使用1000步采样：80.7 Tflops，FID=25.9
- DiT-XL/2 使用128步采样：15.2 Tflops，FID=23.7

**结论**：模型计算量比采样计算量更重要。

---

## 实验设计

### 数据集
- **ImageNet**：256×256和512×512分辨率
- **任务**：类条件图像生成

### 训练配置
- **优化器**：AdamW
- **学习率**：$1 \times 10^{-4}$（常数，无warmup）
- **批大小**：256
- **数据增强**：仅水平翻转
- **EMA**：decay=0.9999
- **硬件**：TPU v3-256 pod

### 评估指标
- **主要指标**：FID-50K（使用250步DDPM采样）
- **次要指标**：Inception Score、sFID、Precision/Recall

### 结果分析

**ImageNet 256×256基准测试**：

| 方法 | FID↓ | IS↑ | Precision | Recall |
|------|------|-----|-----------|--------|
| LDM-4-G (cfg=1.50) | 3.60 | 247.67 | 0.87 | 0.48 |
| StyleGAN-XL | 2.30 | 265.12 | 0.78 | 0.53 |
| **DiT-XL/2-G (cfg=1.50)** | **2.27** | **278.24** | **0.83** | **0.57** |

**ImageNet 512×512基准测试**：

| 方法 | FID↓ | IS↑ |
|------|------|-----|
| ADM-G, ADM-U | 3.85 | 221.72 |
| **DiT-XL/2-G (cfg=1.50)** | **3.04** | **240.82** |

DiT-XL/2在两个分辨率上都达到了**SOTA性能**。

---

## 讨论

### 优势

- **架构统一性**：证明Transformer可以成功替代U-Net，推动生成模型架构统一化
- **优秀的可扩展性**：计算量与性能呈强相关，为大规模模型发展指明方向
- **训练稳定性**：无需特殊技巧即可稳定训练
- **计算效率**：在相近或更少的计算量下达到更好的性能
- **更高的Recall**：相比LDM，DiT在所有guidance scale下都有更高的recall值

### 局限性

- **依赖预训练VAE**：使用Stable Diffusion的VAE，是混合架构而非纯Transformer
- **仅探索类条件生成**：未涉及文生图等更复杂的条件生成任务
- **计算资源需求**：大规模模型训练需要TPU集群
- **patch大小的权衡**：更小的patch提升性能但增加计算量

### 与传统U-Net的对比

| 特性 | U-Net | DiT |
|------|-------|-----|
| 归纳偏置 | 强（卷积、多尺度） | 弱（纯注意力） |
| 可扩展性 | 有限 | 优秀 |
| 架构统一性 | 领域特定 | 跨领域通用 |
| 训练稳定性 | 需要技巧 | 天然稳定 |

---

## 相关工作

### Transformer在生成模型中的应用

- **自回归模型**：GPT系列、ImageGPT、DALL·E
- **掩码生成模型**：MaskGIT、MAGE
- **DALL·E 2**：使用Transformer生成CLIP embedding

### 扩散模型架构

- **DDPM** (Ho et al., 2020)：首次引入U-Net作为扩散模型主干
- **ADM** (Dhariwal & Nichol, 2021)：改进U-Net设计，达到SOTA
- **LDM** (Rombach et al., 2022)：潜在空间扩散模型
- **Concurrent work**：U-ViT探索了类似的Transformer架构

### 架构复杂度分析

- **ViT** (Dosovitskiy et al., 2021)：证明Transformer在视觉任务中的可扩展性
- **Scaling Laws**：语言模型中计算量与性能的幂律关系

---

## 未来方向

1. **扩展到文生图任务**：将DiT应用于DALL·E 2、Stable Diffusion等文生图模型
2. **���大规模的模型**：继续扩展模型规模，探索scaling law
3. **纯Transformer架构**：在像素空间训练DiT，摆脱VAE依赖
4. **多模态条件生成**：探索更复杂的条件注入机制
5. **高效采样方法**：结合DiT开发更快的采样算法
6. **架构搜索**：自动化探索最优的DiT配置

---

## 技术细节补充

### VAE vs VQ-VAE

| 特性 | VAE (DiT使用) | VQ-VAE |
|------|---------------|--------|
| 潜在空间 | 连续（浮点数） | 离散（codebook索引） |
| 输出维度 | 32×32×4（4通道特征） | 32×32（单个索引） |
| 适用场景 | 扩散模型 | 自回归模型 |
| 量化 | 无 | 有（查表） |

### Transformer Block完整结构

```python
def dit_block(x, c):
    # 生成6个调制参数
    γ₁, β₁, α₁, γ₂, β₂, α₂ = adaLN_modulation(c)

    # 子层1: Multi-Head Self-Attention
    x = x + α₁ * Attention(adaLN(x, γ₁, β₁))

    # 子层2: Point-wise Feed-Forward
    x = x + α₂ * FFN(adaLN(x, γ₂, β₂))

    return x

def adaLN(x, γ, β):
    return γ * normalize(x) + β
```

### 计算复杂度分析

对于DiT-XL/2（$N=256$ tokens，$d=1152$）：

- **Self-Attention**：$O(N^2 \cdot d) \approx 75M$ ops
- **Point-wise FFN**：$O(N \cdot d^2 \cdot 2) \approx 680M$ ops

FFN占据了大部分计算量！

---

## 参考文献

- Peebles, W., & Xie, S. (2023). Scalable Diffusion Models with Transformers. ICCV 2023.
- Ho, J., et al. (2020). Denoising Diffusion Probabilistic Models. NeurIPS 2020.
- Rombach, R., et al. (2022). High-Resolution Image Synthesis with Latent Diffusion Models. CVPR 2022.
- Dosovitskiy, A., et al. (2021). An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale. ICLR 2021.
- Dhariwal, P., & Nichol, A. (2021). Diffusion Models Beat GANs on Image Synthesis. NeurIPS 2021.

---

## 关键要点总结

1. **架构创新**：首次系统性地将纯Transformer应用于扩散模型
2. **adaLN-Zero**：零初始化的自适应归一化是性能关键
3. **Gflops定律**：计算量与生成质量强相关（-0.93）
4. **SOTA性能**：ImageNet 256×256达到FID 2.27
5. **统一架构**：推动生成模型向Transformer统一的趋势