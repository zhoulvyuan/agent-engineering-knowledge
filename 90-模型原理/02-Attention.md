# Attention / 注意力机制

> **状态**：🟢 已理解 ｜ **深度**：C 理解概念 ｜ **更新**：2026-09-15
> **上级**：[90-模型原理](README.md) ｜ **依赖**：无

## 一句话定义

> Attention 是一种让模型在生成每个输出时，根据相关性对输入序列中所有位置动态加权求和的机制：每个位置用 Query 去匹配所有位置的 Key，得到权重后对相应的 Value 做加权汇总。

## 解决的问题

> 在 Attention 普及之前，序列建模主要靠 RNN/LSTM，存在三类麻烦：
> - **长距离依赖丢失**：信息要沿时间步逐级传递，距离越远梯度衰减越严重，模型"记不住"开头的信息。
> - **无法并行**：RNN 必须按时间步串行计算，训练慢，难以利用 GPU 的并行能力。
> - **信息瓶颈**：Encoder-Decoder 把整句话压缩成一个固定长度向量，长句子信息被压扁。Attention 让 Decoder 在每一步都能直接"回看"源序列的全部位置，绕开了这个瓶颈。


## 待回答的问题

- [x] Attention 的基本思想
- [x] Self-Attention 在做什么
- [x] 注意力与上下文长度的关系（为什么长上下文很贵）

### Attention 的基本思想

核心是"按相关性检索并加权汇总"，可以类比一次软性的字典查询：

- **三要素**：Query（我要找什么）、Key（每个位置的索引）、Value（每个位置的实际内容）。
- **流程**：用 Query 与每个 Key 做点积得到相似度分数 → 除以 √d_k 缩放 → softmax 归一化成权重 → 用权重对所有 Value 加权求和。公式即 `Attention(Q,K,V) = softmax(QKᵀ/√d_k)V`。
- **"软"检索**：和哈希表的硬查找不同，Attention 返回的是所有 Value 的加权混合，权重连续可导，因此可端到端训练。
- **缩放的作用**：d_k 较大时点积数值会很大，使 softmax 进入饱和区、梯度消失，除以 √d_k 把方差拉回稳定范围。

### Self-Attention 在做什么

Self-Attention（自注意力）指 Q、K、V 都来自**同一个序列**，让序列内部每个 token 根据与其他 token 的关系来重新表示自己：

- **建立 token 间关系**：每个词的新表示是序列中所有词（含自身）的加权组合，权重反映语义/语法相关度，例如代词可以"注意"到它指代的名词。
- **全局感受野、一步到位**：任意两个位置的交互都是直接的 O(1) 路径长度，不像 RNN 要走 O(n) 步，长距离依赖天然好捕捉。
- **多头机制（Multi-Head）**：把表示切成多个子空间并行做 Attention，不同头可关注不同类型的关系（句法、共指、位置等），再拼接融合。
- **需要位置编码**：Self-Attention 本身对顺序不敏感（置换等变），所以必须额外注入位置信息（正弦编码、RoPE、ALiBi 等）才能区分词序。
- 这是 Transformer Encoder/Decoder 的核心算子，也是现代大模型的基础。

### 注意力与上下文长度的关系（为什么长上下文很贵）

标准 Self-Attention 的开销随序列长度 n 呈平方级增长，这是长上下文昂贵的根本原因：

| 维度 | 复杂度 | 说明 |
| --- | --- | --- |
| 计算量 | O(n²·d) | QKᵀ 要算 n×n 的注意力矩阵，n 翻倍计算量变 4 倍 |
| 显存 | O(n²) | 注意力分数矩阵本身随 n² 增长（FlashAttention 可省掉显式存储但计算量不变） |
| 推理 KV Cache | O(n·d·层数) | 自回归生成需缓存历史所有 token 的 K、V，显存随上下文线性增长，常成为长上下文推理的主要瓶颈 |

- **工程缓解手段**：FlashAttention（IO 感知、减少显存读写）、稀疏/滑窗注意力（Longformer、Sliding Window）、线性注意力近似、GQA/MQA（多个 Query 头共享 KV 以压缩 KV Cache）、KV Cache 量化与分页（PagedAttention）。
- 一句话：计算是 O(n²)，而在线推理阶段 KV Cache 带来的 O(n) 显存往往是实际部署中更先触顶的限制。

## 核心内容

- **三要素 Q/K/V**：由输入分别乘可学习的投影矩阵 W_Q、W_K、W_V 得到。
- **缩放点积注意力**：`softmax(QKᵀ/√d_k)V`，是最常用的打分函数（另有加性注意力 Bahdanau）。
- **多头注意力**：并行多组低维 Q/K/V，捕捉多种关系后拼接 + 线性变换。
- **掩码机制**：
  - *因果掩码（Causal Mask）*：解码时屏蔽未来 token，保证自回归不"偷看答案"。
  - *Padding Mask*：屏蔽补齐位置，避免无效 token 干扰。
- **位置编码**：绝对（Sinusoidal/可学习）、相对位置、RoPE、ALiBi，用于注入顺序信息。
- **变体**：Self-Attention（序列内）、Cross-Attention（Decoder 查询 Encoder 输出）、稀疏/局部注意力、线性注意力、GQA/MQA。
- **残差 + LayerNorm**：Attention 子层通常包裹在残差连接和归一化中，构成 Transformer Block。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | 词嵌入 / 位置编码、线性投影与 softmax | Attention 的输入是嵌入向量，依赖位置编码补充顺序，内部核心是矩阵乘法 + softmax 归一化 |
| 下游 | Transformer、BERT/GPT 等大模型、Cross-Attention（含多模态对齐）| Attention 是 Transformer 的核心算子，所有主流 LLM 以其为基础；跨模态模型用它做图文对齐 |
| 易混淆 | 卷积/池化、RNN 的隐藏状态传递、"注意力权重=可解释性" | 卷积是固定局部感受野、Attention 是数据相关的全局加权；RNN 靠串行状态而非一次性全局交互；注意力权重不等于严格的因果解释 |

## 典型工程场景

- **大语言模型**：GPT/Claude/DeepSeek 等的文本生成、对话、代码补全核心即 Self-Attention。
- **机器翻译 / 摘要**：Encoder-Decoder 的 Cross-Attention 实现源语与目标语对齐。
- **检索增强生成（RAG）**：模型通过 Attention 聚合拼接进上下文的检索片段。
- **多模态**：CLIP/图文大模型用 Cross-Attention 对齐图像 patch 与文本 token。
- **代码智能**：IDE 代码补全、函数级理解依赖 Attention 建模长距离符号依赖。
- **语音/时序**：Conformer、时序预测中用注意力捕捉长程模式。

## 适用边界

**适合使用**

- 需要建模长距离依赖、全局上下文关系的任务（语言、代码、图文对齐）。
- 可利用 GPU 大规模并行训练的场景。
- 输入输出之间需要动态、数据相关对齐的任务（翻译、检索汇总）。

**不适合使用**

- 超长序列且算力/显存受限时，标准 Attention 的 O(n²) 开销难以承受（需换稀疏/线性变体或分段处理）。
- 低延迟、极低资源的边缘设备上做短序列任务，轻量 CNN/RNN 可能更划算。
- 数据量极小的任务：Attention/Transformer 参数多、归纳偏置弱，小数据下易过拟合、不如带强先验的模型。

## 常见误区

- **"注意力权重 = 模型的解释"**：权重只是中间加权，不能直接等同于因果归因或可靠的可解释性。
- **"Attention 彻底取代了 RNN/CNN"**：它主要替代了序列建模的主干，但局部/轻量场景 CNN、RNN 仍有价值，且 Attention 常与它们混用。
- **"Self-Attention 天然懂词序"**：它本身对顺序不敏感，必须靠位置编码补充。
- **"上下文越长越好"**：长上下文带来平方级成本与 KV Cache 压力，且模型存在"中间信息丢失"（lost in the middle）问题。
- **"多头就是多算几遍一样的东西"**：各头在不同子空间学习，关注的关系类型不同，并非简单重复。
- **"FlashAttention 把复杂度降到了线性"**：它优化的是显存 IO 和实测速度，计算复杂度仍是 O(n²)。

## 我的结论

> **该用**：当任务依赖长距离、全局、数据相关的上下文关系，且有足够数据和算力时，Attention（Transformer）几乎是默认首选——它是现代 NLP、代码、多模态的事实标准主干。
> **不该用 / 要谨慎**：序列极长且资源紧张时，别无脑上标准 Attention，应评估稀疏/线性注意力、滑窗、GQA/MQA 及 KV Cache 优化；小数据或强局部性、低延迟边缘场景，先考虑更轻、先验更强的模型。
> 一句话：Attention 的价值在于"用可并行的方式一次性建立全局关系"，代价是平方级开销；工程上真正的功夫往往花在如何在长上下文下把这个代价压下来。

## 待解决问题

- [ ] 各类线性/稀疏注意力（Performer、Linear Attention、Mamba 类 SSM）在真实长上下文任务中的质量-效率权衡到底如何，何时能安全替代标准 Attention？
- [ ] RoPE、ALiBi、NTK 插值等位置方案在上下文外推（train-short-test-long）时的边界与失效模式。
- [ ] GQA/MQA、PagedAttention、KV Cache 量化在不同模型规模与并发下的具体收益与精度损失数据。

## 参考资料

- Vaswani et al., "Attention Is All You Need"（2017，Transformer 原始论文）
- Bahdanau et al., "Neural Machine Translation by Jointly Learning to Align and Translate"（2014，加性注意力）
- Dao et al., "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"（2022）
- Ainslie et al., "GQA: Training Generalized Multi-Query Transformer Models"（2023）
- Su et al., "RoFormer: Enhanced Transformer with Rotary Position Embedding"（RoPE）
- Kwon et al., "Efficient Memory Management for LLM Serving with PagedAttention"（vLLM，2023）
- Liu et al., "Lost in the Middle: How Language Models Use Long Contexts"（2023）
- "The Illustrated Transformer"（Jay Alammar，图解入门博客）
