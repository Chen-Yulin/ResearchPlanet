---
id: MotionGPT3
tags:
  - Diffusion
  - Transformer
  - MultiModal
  - MotionGeneration
  - MoT
  - VAE
date: 2026-04-07 00:00:00
categories: Review
publish: 2025 Nov
---
# MotionGPT3 - Human Motion as a Second Modality

[论文 arXiv](https://arxiv.org/abs/2506.24086) | [GitHub](https://github.com/OpenMotionLab/MotionGPT3)

---
## 研究背景
文本（Text）是离散符号，动作（Motion）是连续信号，两者性质截然不同。现有方法面临两个核心矛盾：
- **量化误差（Quantization Error）**：将动作离散化为 VQ 码本索引以适配 LLM 的 next-token prediction，会引入近似误差，衰减高频细节，破坏语义-物理一致性
- **跨模态干扰（Cross-Modal Interference）**：在单流（Single-Stream）骨架中混合文本与动作 token，梯度相互拉扯，导致训练不稳定、收敛慢

---
## 研究目标
提出一个统一的动作-语言模型，同时支持：
- **Text-to-Motion（T2M）**：文本描述 → 生成动作
- **Motion-to-Text（M2T）**：动作序列 → 生成文本描述

且需要避免量化误差，减少跨模态干扰。

---
## 核心概念
### Mixture-of-Transformers（MoT）
来自 Liang et al. (2024) 的思想：为每个模态配备独立的 Transformer 分支，各自拥有独立的 Embedding、FFN 和 LayerNorm，**仅在 Self-Attention 层共享**。这样：
- 各模态保留自身的归纳偏置（Inductive Bias）
- 跨模态信息仅通过 Attention 受控交换
- 新增模态只需新增分支，无需重训全部参数
### 连续动作表示（Continuous Motion Representation）
使用预训练的 Motion VAE 将动作编码为连续 latent 向量（而非 VQ 离散索引），避免量化误差。

---
## 研究方法
### 整体架构
```
                Motion VAE                    Bimodal LLM                   Motion VAE
              ┌───────────┐            ┌────────────────────┐           ┌───────────┐
原始动作 m ───→│ Encoder E  │──→ z₀     │ Text Branch T      │           │ Decoder D │──→ 动作
              │ 9层Trans.  │           │ Motion Branch M    │           │ 9层Trans. │
              │ 4头,skip   │           │ + Diffusion Head H │──→ ẑ₀ ──→│ 4头,skip  │
              └───────────┘            └────────────────────┘           └───────────┘
```
### 1. Motion VAE
- 来自 Xin et al. (2023)（MLD）
- Encoder $\mathcal{E}$: 将 $N$ 帧动作 $m^{1:N}$ 编码为单个 latent $z \in \mathbb{R}^{256}$
- Decoder $\mathcal{D}$: 将 latent $z$ 解码回动作序列
- 训练目标：重建损失 + KL 正则
- **预训练后冻住**，不参与后续训练
- 动作长度信息隐式编码在 latent 中，Decoder 可生成变长输出
### 2. 双分支 Bimodal Transformer（MoT 架构）
两个并行分支，基于 GPT-2 配置（12 层，维度 768，MLP 维度 3072）：

| 组件 | Text Branch $\mathcal{T}$ | Motion Branch $\mathcal{M}$ |
|---|---|---|
| 初始化 | 预训练 GPT-2（124M） | 从零训练（238M） |
| Embedding | 文本 Embedding | 独立 Motion Embedding |
| FFN | 独立 | 独立 |
| LayerNorm | 独立 | 独立 |
| **Self-Attention** | **共享** | **共享** |

**路由机制**：硬路由（非学习），由特殊标记决定——
- `<som>` / `<eom>` 界定动作边界
- `<motion_in>` / `<motion_out>` 标记 I/O 位置
- 文本 token（$\vartheta_i = 0$）→ Text Branch
- 动作 token（$\vartheta_i = 1$）→ Motion Branch
### 3. 动作接口模块
由于动作是连续信号，不能复用文本的 Embedding lookup 和 softmax 解码，需要专用接口：
- **MUH（Motion Understanding Head）**：线性投影，将 VAE latent 映射到 Transformer 输入空间（理解任务）
- **MGH（Motion Generation Head）**：即 Diffusion Head $\mathcal{H}$，将 Transformer 隐状态映射回 VAE latent 空间（生成任务）
### 4. Diffusion Head $\mathcal{H}$
轻量级扩散模型（3 层 MLP + ResBlock，隐藏维度 1024），在 VAE latent 空间中做去噪：
$$\mathcal{L}_{\text{diff}} = \mathbb{E}_{z_0, t, \epsilon}\left[\|\epsilon - \mathcal{H}(z_t, h_m)\|_2^2\right]$$
- 训练时：对 $z_0$ 加噪得 $z_t$，$\mathcal{H}$ 以 Motion Branch 隐状态 $h_m$ 为条件，学习预测噪声 $\epsilon$
- 推理时：插入 $K$ 个 `<motion_out>` 占位 token，Motion Branch 输出 $h_m^{v:v+K}$，$\mathcal{H}$ 从纯噪声 $z_T$ 逐步去噪（默认 100 步）得到 $\hat{z}_0$，再由 VAE Decoder 解码

---
## 训练策略：三阶段对齐
### Stage I: Text-to-Motion 预训练
- **冻住** Text Branch $\mathcal{T}$
- 只训练 Motion Branch $\mathcal{M}$ + Diffusion Head $\mathcal{H}$
- 任务：T2M（文本→动作生成）
- 目的：让 $\mathcal{M}$ 在稳定的语言条件下学会动作语义
- 100k iterations
### Stage II: 跨模态对齐（Cross-Modal Alignment）
- 仍**冻住** $\mathcal{T}$
- 引入多任务：T2M + M2T + Motion Prediction
- 以指令格式呈现，促进双向对齐
- 300k iterations
### Stage III: 联合微调（Joint Fine-Tuning）
- **解冻** $\mathcal{T}$，全参数微调
- 混合文本-动作配对数据 + 纯文本数据
- 50k iterations

---
## 主要发现
### 双流 vs 单流
- 双流架构收敛速度约为单流的 **2 倍**（训练损失）
- 验证指标（R@3, MMDist）收敛快 **4 倍**
- 相同损失水平下，双流模型质量更优
### 连续 VAE vs 离散 VQ
- VQ 方案在 R@3 约 0.5 时即饱和（天花板低）
- VAE 连续表示持续改进，最终质量显著更高
### 实验结果
在 HumanML3D 上：
- T2M：R@3 = 0.837，MMDist = 2.725，达到 SOTA
- M2T：BertScore = 35.231，超越现有统一模型

---
## 讨论
### 优势
- 避免 VQ 量化误差，保留高频运动细节
- 双分支设计减少梯度干扰，加速收敛
- 三阶段训练抑制跨任务干扰
- 仅需 2 张 3090，训练高效
### 局限性
- VAE 输出单个 latent，**不支持长动作的分段组合生成**
- 方向性控制（左/右）有时会失败
- 泛化能力受限于数据覆盖范围

---
## 与其他多模态架构的对比

| 架构类型 | 路由方式 | 代表 | 特点 |
|---|---|---|---|
| 单流 + Projector | 无路由，全拼接 | LLaVA, Qwen-VL | 简单，但有跨模态干扰 |
| MoE | 学习的 Router, TopK 选专家 | Mixtral, Switch Transformer | 动态路由，扩展性好 |
| MoT / 双分支 | 按模态硬路由，共享 Attention | **MotionGPT3** | 隔离前馈，受控交互 |

---
## 未来方向
1. **分层/分段 latent**：用 hierarchical 或 segment-wise latent 表示支持长动作和组合生成
2. **更大数据集和更强 LLM**：扩展训练规模，评估效率和鲁棒性
3. **局部语义对齐**：支持段级别的文本-动作精细对应

---
## 参考文献
- Zhu, B., Jiang, B., Wang, S., et al. (2025). MotionGPT3: Human Motion as a Second Modality. arXiv:2506.24086.
- Xin, T., et al. (2023). MLD: Motion Latent Diffusion.
- Liang, C., et al. (2024). Mixture-of-Transformers (MoT).
- Radford, A., et al. (2019). GPT-2.
