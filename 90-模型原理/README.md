# 90 模型原理

> 支撑域 ｜ 本域不作为主要学习路线，而是在 AI 工程能力建立后逐步深入。

## 本域定位

本域遵循**贯穿式补充**的学习方式：

```text
遇到实际问题
   ↓
发现需要模型原理
   ↓
补充 Transformer / Attention / Training / Inference 等知识
```

**不要按传统学院派顺序学这一域。** 它是你的"知识储备库"而非"学习任务"。
典型触发场景：

| 你遇到的问题 | 需要补的原理 |
| --- | --- |
| 为什么长上下文这么贵 | Attention |
| 为什么 temperature=0 也不完全确定 | Inference |
| 这个场景该微调还是该用 RAG | Fine-tuning |
| 该不该自建模型服务 | Model Serving / Quantization |
| 模型为什么会产生幻觉 | Pre-training / Inference |

**结构决策**：原地图中 `1.4 Transformer` 与 `10.1 / 10.2` 重复，
已统一到本域。

## 知识点清单

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| [Transformer](01-Transformer.md) | ⚪ | C | 现代大模型的骨架 |
| [Attention / 注意力机制](02-Attention.md) | ⚪ | C | 上下文如何被模型使用 |
| [Pre-training / 预训练](03-Pre-training.md) | ⚪ | C | 模型能力的来源 |
| [Fine-tuning / 微调](04-Fine-tuning.md) | ⚪ | C | 用数据改造模型行为 |
| [RLHF / 偏好优化](05-RLHF与偏好优化.md) | ⚪ | C | 让模型输出符合人类偏好 |
| [Inference / 推理](06-Inference.md) | ⚪ | C | 模型如何生成回答 |
| [Quantization / 量化](07-Quantization.md) | ⚪ | D | 用精度换成本的部署手段 |
| [Model Serving / 模型服务](08-Model-Serving.md) | ⚪ | D | 模型的生产部署 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 01-基础认知 | 本域是其原理层 |
| 10-技术选型 | 模型选型需要原理支撑 |
| 08-Agent工程化 | 成本与延迟的根本原因在本域 |

## 建议学习顺序

按需，而非按序。

如果一定要给顺序：**Inference → Attention → Transformer → Fine-tuning → 其余**。

理由：`Inference` 解释了"模型是怎么一个字一个字吐出来的"，这一条工程价值最高，
也最容易理解；`Fine-tuning` 关系到选型决策；`Quantization` / `Model Serving`
除非你要自建服务，否则可以一直保持 ⚪。

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 建立本域骨架 |
