---
id: ChemGPT
tags:
  - Chemistry
  - LLM
date: 2025-11-27 22:51:57
categories: Review
publish: "2022"
---
## Base Model
ChemGPT 早期基座模型基于 EleutherAI 的语言模型 GPT‑Neo，在分子字符串语料（SMILES 或 SELFIES）上进行自回归建模。
典型预训练数据集为公开的大规模分子库 PubChem10M，参数规模覆盖百万到十亿级别（如约 4.7M 的轻量版本或 100M+ 的科研版本）。
属于领域特化的 decoder-only 化学语言模型。

## Training Method
- **对分子进行编码**：对 SMILES/SELFIES/反应式进行 tokenization，构建化学“语言”词表（原子、键、环、分支、立体标记等均被离散化为 token）
- **预训练目标**：自回归分子语言建模（Next-token prediction），最大化生成合法分子序列的对数似然。

## 调用方式
### 网页直接使用
https://www.chemgpt.app/
（我没有注册账号，直接问会显示网络繁忙）
![[Pasted image 20251127230555.png]]
### Huggingface 下载模型并调用
当前 HF Hub 上最常被引用的 ChemGPT 版本包括：
- 轻量模型：ChemGPT 4.7M （https://huggingface.co/ncfrey/ChemGPT-4.7M）
- 中型模型：ChemGPT 1.2B （https://huggingface.co/ncfrey/ChemGPT-1.2B）

#### **模型推理（文本生成 Pipeline）**
`text-generation pipeline` 适合生成式任务（包括化学问答、分子生成）。
```python
from transformers import pipeline

chem_pipe = pipeline(
    "text-generation",
    model="ncfrey/ChemGPT-1.2B",  # 或 ncfrey/ChemGPT-4.7M
    device_map="auto"
)

prompt = "CCO>>"  # 以乙醇为前缀生成分子衍生结构或产物候选
result = chem_pipe(
    prompt,
    max_new_tokens=128,
    temperature=0.7,
    do_sample=True
)

print(result[0]["generated_text"])

```

#### **模型推理（直接使用 model.generate）**
当需要精细控制 token 级生成时，直接调用 `generate API`。
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = "ncfrey/ChemGPT-1.2B"  # 或 Chemistry 结构生成模型 ncfrey/ChemGPT-4.7M
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.bfloat16)
model.eval()
model.to("cuda" if torch.cuda.is_available() else "cpu")

prompt = "CCO>>"
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

with torch.no_grad():
    output_ids = model.generate(
        **inputs,
        max_new_tokens=100,
        do_sample=True,
        temperature=0.6,
        top_p=0.95
    )

generated = tokenizer.decode(output_ids[0], skip_special_tokens=True)
print(generated)

```
