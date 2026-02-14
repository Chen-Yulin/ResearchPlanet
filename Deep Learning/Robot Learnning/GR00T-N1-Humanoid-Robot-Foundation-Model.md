---
id: GR00T N1 An Open Foundation Model for Generalist Humanoid Robots
tags:
  - RobotLearning
  - FoundationModel
  - VLA
  - HumanoidRobot
  - ImitationLearning
  - DiffusionModel
  - Robotics
  - VLM
  - Multi-modal
  - Transformer
date:
  - 2026-02-02 11:30:00
categories: Review
---
![[Pasted image 20260203120646.png]]
# GR00T N1: 通用人形机器人开放基础模型

[论文链接](https://arxiv.org/abs/2503.14734v2) | NVIDIA, 2025

---

## 研究背景

人形机器人作为通用机器人的理想硬件平台，需要强大的基础模型来实现智能自主操作。受大语言模型和视觉模型成功的启发，研究者希望通过在大规模异构数据上训练机器人基础模型，使其能够理解新场景、处理真实世界的变化并快速学习新任务。然而，与文本和图像领域不同，机器人领域缺乏互联网规模的训练数据，不同机器人的传感器、自由度、控制模式差异巨大，形成"数据孤岛"问题。

---

## 研究目标

本论文要解决的核心问题：

1. **数据稀缺问题**：人形机器人数据收集成本高、耗时长，如何突破真实数据瓶颈
2. **跨具身泛化**：如何统一不同机器人的状态和动作空间，实现跨具身学习
3. **数据效率**：如何在有限数据下快速适应新任务并在真实环境中鲁棒执行
4. **端到端优化**：如何将高层推理与低层控制统一到单一模型中

---

## 核心概念

### Vision-Language-Action (VLA) 模型

视觉-语言-动作模型，接收图像观察和语言指令作为输入，直接输出机器人动作。与传统的分层方法（VLM规划 + 低层策略执行）不同，VLA模型实现端到端优化。

### 双系统架构 (Dual-System Architecture)

受人类认知理论启发（Kahneman, 2011），将模型分为：
- **System 2（推理系统）**：慢速、深思熟虑的高层推理
- **System 1（反应系统）**：快速、自动化的低层控制

### 数据金字塔 (Data Pyramid)

将异构训练数据按规模和具身特异性组织成三层结构：
- **底层**：大规模网络数据和人类视频（通用先验）
- **中层**：合成数据（仿真+神经生成，可扩展）
- **顶层**：真实机器人数据（具身特定，高质量）

### 潜在动作 (Latent Actions)

通过VQ-VAE([[VQ-VAE-and-Latent-Action-for-Robotics]])学习的通用动作表示，能够统一不同具身体（包括人类）的动作空间，使无动作标签的视频数据可用于训练。

---

## 研究方法

### 模型架构

GR00T N1采用双系统组合架构，总参数量22亿（GR00T-N1-2B）：

#### System 2: Vision-Language Module

```
输入处理:
├─ 图像: SigLIP-2编码器 → 64个token (224×224)
└─ 文本: SmolLM2 tokenizer → 文本token

特征提取:
└─ Eagle-2 VLM (1.34B参数)
   ├─ 处理vision-language tokens
   └─ 输出: 中间层embeddings φ_t (第12层)
```

**关键设计**：
- 使用中间层而非最终层特征（更快推理+更高成功率）
- 语言组件冻结（保留预训练知识）
- 视觉编码器可训练（适应机器人任务）
- 运行频率：10Hz

#### System 1: Diffusion Transformer Module

```
DiT Block结构（重复N次）:
├─ Self-Attention
│  └─ 输入: noised action tokens + state embeddings
│
└─ Cross-Attention
   ├─ Query: action/state tokens
   └─ Key & Value: VLM输出的φ_t
```

**动作生成流程**：
1. 输入加噪动作 $A_t^{\tau} = \tau A_t + (1-\tau)\epsilon$，其中 $\tau \in [0,1]$
2. 通过DiT迭代去噪（K=4步）
3. 输出16步动作序列（action chunking）
4. 运行频率：120Hz

**Flow-Matching损失**：

$$
\mathcal{L}_{fm}(\theta) = \mathbb{E}_{\tau} \|V_{\theta}(\varphi_t, A_t^{\tau}, q_t) - (\epsilon - A_t)\|^2
$$

其中 $V_{\theta}$ 是[[Diffusion-Transformers-DiT]]模型，预测去噪向量场。

#### 模块交互机制

```
信息流:
图像 + 语言指令
    ↓
[System 2: Eagle-2 VLM]
    ↓ (输出 φ_t)
[Cross-Attention Bridge]
    ↓
[System 1: DiT]
├─ Self-Attention (action + state)
└─ Cross-Attention (attend to φ_t)
    ↓
16步动作序列
```

**端到端联合训练**：
- 两个模块通过cross-attention紧密耦合
- 使用统一的flow-matching loss优化
- 辅助目标检测loss增强空间理解：

$$
\mathcal{L} = \mathcal{L}_{fm} + \mathcal{L}_{det}
$$

### 异构数据训练策略

#### 1. 数据金字塔组织

| 层级 | 数据源 | 时长 | 特点 |
|------|--------|------|------|
| 顶层 | 真实机器人数据 | 3,289小时 | 具身特定，高质量 |
| 中层 | 仿真数据 | 1,743小时 | 可扩展，物理约束 |
| 中层 | 神经生成数据 | 827小时 | 反事实场景，多样性 |
| 底层 | 人类视频 | 2,517小时 | 大规模，通用先验 |

**总计**：8,376小时训练数据

#### 2. 潜在动作学习

**VQ-VAE训练**：

```python
# 编码器
输入: (当前帧 x_t, 未来帧 x_{t+H})
     ↓
Encoder → 连续embedding → 量化到codebook
     ↓
潜在动作 z_t

# 解码器
输入: x_t + z_t
     ↓
Decoder → 重建 x_{t+H}
```

**跨具身一致性**：
- 同一潜在动作在不同具身体中语义一致
- 例如：潜在动作1 = "右臂向左移动"（对所有机器人和人类）

**训练使用**：
- 提取预量化连续embedding作为"LAPA具身体"的动作
- 使用flow-matching loss训练

#### 3. 神经轨迹生成

**目标**：从88小时真实数据扩增到827小时（~10倍）

**技术流程**：

```
步骤1: 微调视频生成模型
├─ 基础模型: WAN2.1-I2V-14B
├─ 方法: LoRA微调
├─ 数据: 3,000条轨迹，81帧@480P
└─ 训练: 100 epochs

步骤2: 生成反事实轨迹
├─ 输入: 初始帧 + 新语言指令
├─ 语言生成: 多模态LLM检测物体
│   生成"pick {object} from {A} to {B}"
└─ 输出: 高质量视频

步骤3: 质量过滤
├─ 采样8帧 → LLM判断是否遵循指令
└─ 不合格 → 重新标注

步骤4: 动作标注
├─ 潜在动作编码器 → LAPA
└─ 逆动力学模型 → 伪动作标签
```

**生成能力**：
- 改变操作手（左手↔右手）
- 改变目标位置和物体
- 处理仿真难题（液体、铰接物体）
- 多视角生成（4宫格视频）

#### 4. 仿真数据自动生成

**DexMimicGen系统**：

```
输入: 少量人类演示（几十条）
     ↓
分割 → 物体中心的子任务片段
     ↓
变换 → 根据新物体位置调整
     ↓
组合 → 插值并组合片段
     ↓
验证 → 仿真执行，保留成功轨迹
     ↓
输出: 每任务10,000条演示
```

**规模**：
- 54个源-目标容器组合
- 540,000条预训练轨迹
- 11小时生成 = 6,500小时等效人类演示

#### 5. 具身特定编码器/解码器

**处理不同维度的状态和动作**：

```python
embodiments = {
    "GR-1": {
        "state": [joint_pos, joint_vel, base_pos, ...],
        "action": [joint_targets, ...],
        "encoder": MLP_GR1,
        "decoder": MLP_GR1
    },
    "Franka": {
        "state": [ee_pos, ee_rot, gripper],
        "action": [ee_delta, gripper_cmd],
        "encoder": MLP_Franka,
        "decoder": MLP_Franka
    },
    "LAPA": {  # 潜在动作
        "action": [latent_embedding],
        "encoder": MLP_LAPA,
        "decoder": MLP_LAPA
    }
}
```

#### 6. 统一训练框架

**预训练阶段**：
- 全局batch size: 16,384
- 训练步数: 200,000
- 数据混合采样：真实机器人(40%) + 仿真(30%) + 神经(20%) + 人类视频(10%)
- 计算资源: 最多1024个H100 GPU，约50,000 GPU小时

**后训练阶段**：
- Batch size: 128-1024
- 训练步数: 20,000-60,000
- 可选神经轨迹协同训练（1:1采样比例）
- 可在单个A6000 GPU上微调

---

## 主要发现

   ### 预训练泛化能力

在GR-1人形机器人上的零样本评估：

| 任务 | 成功率 | 说明 |
|------|--------|------|
| 左手抓取→右手交接→放置 | 76.6% | 需要双手协调 |
| 新物体→新容器 | 73.3% | 泛化到未见物体 |

### 仿真基准测试

**100条演示/任务的性能对比**：

| 方法 | RoboCasa | DexMG | GR-1 | 平均 |
|------|----------|-------|------|------|
| BC-Transformer | 26.3% | 53.9% | 16.1% | 26.4% |
| Diffusion Policy | 25.6% | 56.1% | 32.7% | 33.4% |
| **GR00T-N1-2B** | **32.1%** | **66.5%** | **50.0%** | **45.0%** |

**关键观察**：
- GR00T N1在所有基准上均优于基线
- 在GR-1任务上优势最明显（+17.3%）

### 真实世界部署

**GR-1人形机器人任务成功率**：

| 任务类型 | Diffusion Policy<br>(10%数据) | Diffusion Policy<br>(全量数据) | GR00T-N1-2B<br>(10%数据) | GR00T-N1-2B<br>(全量数据) |
|---------|---------------------------|---------------------------|---------------------|---------------------|
| 抓取放置 | 3.0% | 36.0% | **35.0%** | **82.0%** |
| 铰接物体 | 14.3% | 38.6% | **62.0%** | **70.9%** |
| 工业操作 | 6.7% | 61.0% | **31.0%** | **70.0%** |
| 多机协作 | 27.5% | 62.5% | **50.0%** | **82.5%** |
| **平均** | **10.2%** | **46.4%** | **42.6%** | **76.8%** |

**数据效率**：
- GR00T N1用10%数据（42.6%）≈ Diffusion Policy用全量数据（46.4%）
- 展现出色的样本效率

### 神经轨迹增强效果

**RoboCasa基准（协同训练3K神经轨迹/任务）**：

| 数据量 | 仅真实数据 | +LAPA | +IDM |
|--------|-----------|-------|------|
| 30条   | 17.4%     | 20.8% (+3.4%) | 20.0% (+2.6%) |
| 100条  | 32.1%     | 38.5% (+6.4%) | 40.9% (+8.8%) |
| 300条  | 49.6%     | 53.8% (+4.2%) | 56.4% (+6.8%) |

**真实世界（协同训练100神经轨迹/任务）**：
- 平均提升：+5.8%

**观察**：
- 低数据场景：LAPA略优（更通用的先验）
- 高数据场景：IDM更优（更接近真实动作）

### 定性分析

**运动质量**：
- GR00T N1运动更流畅，抓取精度更高
- Diffusion Policy常出现初始帧不动、抓取不准确

**泛化能力**：
- 预训练模型能执行未见过的双手交接任务
- 后训练模型在特定任务上更精确，但失去部分泛化能力

---

## 实验设计

### 仿真基准

**RoboCasa Kitchen（24任务）**：
- 机器人：Franka Emika Panda
- 任务：抓取放置、开关门、按按钮、转水龙头等
- 观察：3个RGB相机（左、右、腕部）
- 动作：末端执行器相对位姿 + 夹爪状态
- 数据：每任务3,000条MimicGen生成的演示

**DexMimicGen Cross-Embodiment Suite（9任务）**：
- 具身体：
  - 双臂Panda + 平行夹爪（穿线、组装、运输）
  - 双臂Panda + 灵巧手（清理、抬托盘）
  - GR-1人形 + 灵巧手（倒水、咖啡、分类）
- 数据：每任务1,000条演示

**GR-1 Tabletop Tasks（24任务）**：
- 机器人：GR-1人形 + Fourier灵巧手
- 任务：18个重排任务 + 6个铰接物体任务
- 观察：头部自我中心相机
- 动作：关节位置/旋转 + 腰部/颈部
- 数据：每任务1,000条DexMimicGen生成

### 真实世界基准

**任务类别**：

1. **抓取放置（5任务）**：
   - 托盘→盘子、砧板→篮子、餐垫→碗等
   - 评估：见过和未见过物体

2. **铰接物体（3任务）**：
   - 白色抽屉、深色柜子、木箱
   - 要求：放入物体并关闭

3. **工业操作（3任务）**：
   - 机械零件打包
   - 网格杯倾倒
   - 圆柱体交接

4. **多机协作（2任务）**：
   - 第1部分：抓取→放入网格杯→交给另一机器人
   - 第2部分：接收→放入黄色箱→倾倒剩余物

**数据收集**：
- 遥操作时长：15分钟-3小时/任务
- 过滤低质量轨迹

### 评估协议

**仿真**：
- 每任务100次试验
- 取最后5个checkpoint的最大值
- Checkpoint间隔：500步

**真实机器人**：
- 每任务10次试验（机械打包任务5次）
- 部分评分系统（捕捉不同执行阶段）
- 低数据场景：10%数据子采样

### 训练配置

**预训练**：
- 学习率：1e-4
- 优化器：AdamW (β1=0.95, β2=0.999)
- 学习率调度：cosine，warmup比例0.05
- Batch size：16,384
- 步数：200,000

**后训练**：
- Batch size：128-1024
- 步数：20,000-60,000
- 其他超参数同预训练

---

## 讨论

### 优势

1. **统一的跨具身学习**：
   - 单一模型支持从桌面机械臂到双臂人形机器人
   - 潜在动作空间统一不同具身体

2. **卓越的数据效率**：
   - 10%数据达到基线全量数据性能
   - 预训练提供强大的先验知识

3. **可扩展的数据生成**：
   - 神经轨迹生成：10倍数据扩增
   - 仿真自动生成：11小时生成6,500小时等效数据

4. **端到端优化**：
   - VLM推理与DiT控制联合训练
   - 避免分层方法的接口问题

5. **开源生态**：
   - 公开22亿参数模型
   - 提供训练数据和仿真基准

### 局限性

1. **任务范围限制**：
   - 当前主要关注短时域桌面操作
   - 未涉及长时域移动操作（loco-manipulation）

2. **合成数据质量**：
   - 视频生成模型仍面临多样性和物理一致性挑战
   - 需要质量过滤和重新标注

3. **硬件依赖**：
   - 需要高端GPU进行训练（H100集群）
   - 推理需要L40 GPU（63.9ms/16动作）

4. **泛化-专精权衡**：
   - 后训练提升特定任务性能但损失部分泛化能力
   - 预训练模型能执行双手交接，后训练模型失去此能力

5. **视觉-语言骨干限制**：
   - 当前VLM的空间推理和语言理解能力仍有提升空间
   - 更强的VLM可能进一步提升性能

---

## 相关工作

### 机器人基础模型

**VLA模型**：
- **RT-1/RT-2** (Brohan et al., 2022, 2023)：早期VLA模型，使用Transformer架构
- **π0** (Black et al., 2024)：使用mixture-of-experts连接VLM和动作生成
- **Octo** (Octo Model Team et al., 2024)：跨具身模型，但不微调VLM
- **GR-2** (Cheang et al., 2024)：视频-语言-动作模型

**GR00T N1的区别**：
- 使用简单的cross-attention而非MoE
- 端到端微调VLM视觉编码器
- 支持潜在动作和IDM伪动作

### 机器人数据集

**真实机器人数据**：
- **Open X-Embodiment** (2024)：跨具身数据集联盟
- **AgiBot-Alpha** (2025)：100个机器人的大规模数据集
- **遥操作系统**：VIVE、Apple Vision Pro、Leap Motion

**人类视频数据**：
- **Ego4D** (Grauman et al., 2022)：大规模自我中心视频
- **EPIC-KITCHENS** (Damen et al., 2018)：厨房活动
- **Assembly-101** (Sener et al., 2022)：组装任务

**GR00T N1的创新**：
- 数据金字塔组织而非简单混合
- 潜在动作统一有/无标签数据

### 合成数据生成

**仿真数据**：
- **MimicGen** (Mandlekar et al., 2023)：演示变换和重放
- **DexMimicGen** (Jiang et al., 2024)：灵巧操作数据生成
- **RoboCasa** (Nasiriany et al., 2024)：厨房环境仿真

**神经生成**：
- **视频生成模型**：Sora (Brooks et al., 2024)、WAN (Wan Team, 2025)
- **数据增强**：GenAug (Chen et al., 2023)使用扩散模型增强

**GR00T N1的规模**：
- 827小时神经轨迹（前所未有）
- 540K仿真轨迹（11小时生成）

---

## 未来方向

1. **长时域移动操作**：
   - 扩展到全身运动和导航
   - 需要改进硬件、模型架构和训练数据

2. **更强的视觉-语言骨干**：
   - 提升空间推理能力
   - 增强语言理解和任务规划

3. **改进合成数据生成**：
   - 提高视频生成的多样性和反事实能力
   - 增强物理一致性和真实感
   - 探索自动化初始帧生成（img2img扩散）

4. **新型模型架构**：
   - 探索更高效的推理-控制耦合方式
   - 研究分层时间建模

5. **鲁棒性和泛化**：
   - 提升对环境变化的适应能力
   - 增强零样本和少样本学习能力

6. **多模态感知**：
   - 整合触觉、力觉等其他传感器
   - 探索多模态融合策略

7. **长时域视频生成**：
   - 多轮视频生成实现长任务序列
   - 原子任务组合

---

## 参考文献

- NVIDIA (2025). GR00T N1: An Open Foundation Model for Generalist Humanoid Robots. arXiv:2503.14734v2.
- Black et al. (2024). π0: A vision-language-action flow model for general robot control. arXiv:2410.24164.
- Brohan et al. (2022). RT-1: Robotics transformer for real-world control at scale. arXiv:2212.06817.
- Brohan et al. (2023). RT-2: Vision-language-action models transfer web knowledge to robotic control. arXiv:2307.15818.
- Chi et al. (2024). Diffusion Policy: Visuomotor policy learning via action diffusion. IJRR.
- Jiang et al. (2024). DexMimicGen: Automated data generation for bimanual dexterous manipulation via imitation learning. CoRL.
- Mandlekar et al. (2023). MimicGen: A data generation system for scalable robot learning using human demonstrations. CoRL.
- Nasiriany et al. (2024). RoboCasa: Large-scale simulation of everyday tasks for generalist robots. RSS.
- Open X-Embodiment Collaboration et al. (2024). Open X-Embodiment: Robotic learning datasets and RT-X models.
- Ye et al. (2025). Latent action pretraining from videos. ICLR.
- Kahneman (2011). Thinking, Fast and Slow. Farrar, Straus and Giroux.

---

## 关键代码和资源

- **模型权重**：[HuggingFace](https://huggingface.co/nvidia/groot-n1-2b)
- **训练数据**：[HuggingFace Datasets](https://huggingface.co/datasets/nvidia/groot-n1-data)
- **仿真基准**：[GitHub](https://github.com/NVlabs/GR00T)
- **数据格式**：基于LeRobot格式扩展
- **训练基础设施**：NVIDIA OSMO编排平台

---

## 技术细节补充

### 动作空间标准化

**统一不同具身体的表示**：
- 末端执行器旋转状态：6D旋转表示
- 末端执行器旋转动作：轴角表示
- 位置和关节：Min-max归一化
- 顺序：左臂→右臂，旋转→位置→夹爪

### 辅助目标检测损失

使用OWL-v2检测器标注目标物体边界框：

$$
\mathcal{L}_{det} = \|\mathbf{x}_{pred} - \mathbf{x}_{gt}\|^2
$$

其中 $\mathbf{x}$ 是归一化的边界框中心坐标。

### 推理性能

- **GR00T-N1-2B**：63.9ms采样16步动作（L40 GPU，bf16）
- **VLM频率**：10Hz
- **动作输出频率**：120Hz
- **去噪步数**：K=4

### 计算资源

- **预训练**：最多1024个H100 GPU，约50,000 GPU小时
- **神经轨迹生成**：3,600个L40 GPU，约105K GPU小时（1.5天）
- **后训练**：单个A6000 GPU可微调（仅adapter层时batch size可达200）
