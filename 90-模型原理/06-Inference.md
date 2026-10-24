# Inference / 推理

> **状态**：🟢 已理解 ｜ **深度**：C 理解概念 ｜ **更新**：2026-09-15
> **上级**：[90-模型原理](README.md) ｜ **依赖**：无

## 一句话定义

> 推理（Inference）是把已经训练好的模型权重加载起来、对用户输入执行一次前向计算、从而逐 Token 生成输出文本的过程——即"用模型"而非"训练模型"的运行阶段。

## 解决的问题

> 训练只产出一堆静态权重，本身不能直接服务用户。推理阶段要解决的是：如何在可接受的延迟与成本下，把这堆权重变成一个能实时响应、持续吞吐大量并发请求的在线服务。在它被认真对待之前，人们常把"模型跑得通"等同于"模型能上线"，结果遇到首 Token 延迟高、显存被 KV Cache 撑爆、GPU 利用率低下、单请求成本不可控等一系列工程麻烦。推理工程就是专门解决"让训练好的模型高效、稳定、便宜地对外提供服务"这个问题。


## 待回答的问题

- [x] 推理的完整流程
- [x] Prefill 与 Decode 的区别
- [x] 为什么输出是流式且逐 Token 生成的
- [x] 推理成本由什么决定

### 推理的完整流程

一次典型的 LLM 推理请求经过以下阶段：

1. **分词（Tokenize）**：把输入文本切成 Token id 序列。
2. **Prefill（预填充）**：模型一次性并行处理整段 prompt，计算所有输入 Token 的注意力表示，并把每层每个 Token 的 Key/Value 写入 KV Cache。这一步产出第一个输出 Token。
3. **Decode（解码/自回归生成）**：逐个生成后续 Token。每一步只把"上一个生成的 Token"喂进模型，利用 KV Cache 复用历史，计算下一个 Token 的概率分布。
4. **采样（Sampling）**：按 temperature / top-p / top-k 等策略从概率分布中选出一个 Token。
5. **循环与终止**：把新 Token 追加回序列，重复 Decode，直到遇到 EOS、达到 max_tokens 或命中停止词。
6. **反分词（Detokenize）与返回**：把 Token id 转回文本，流式或整体返回给调用方。

关键点：Prefill 是一次性的并行计算，Decode 是串行的循环，二者计算特征差异很大，是所有推理优化的分水岭。

### Prefill 与 Decode 的区别

| 维度 | Prefill（预填充） | Decode（解码） |
| --- | --- | --- |
| 处理对象 | 整段 prompt 的全部 Token | 每步 1 个新 Token |
| 并行度 | 高，所有输入 Token 并行计算 | 低，严格自回归串行 |
| 瓶颈类型 | 计算密集（compute-bound），吃算力 FLOPs | 访存密集（memory-bound），吃显存带宽 |
| 耗时体现 | 决定首 Token 延迟 TTFT | 决定每 Token 间隔 TPOT / 整体吞吐 |
| 随长度增长 | 随 prompt 长度近似二次（注意力）/ 线性增长 | 每步成本随已生成上下文线性增长 |

工程含义：Prefill 阶段 GPU 算力是瓶颈，适合大 batch 提高利用率；Decode 阶段显存带宽是瓶颈，GPU 算力常常闲置，所以需要 continuous batching、投机解码等手段把串行步"填满"。现代推理框架（如 vLLM、TensorRT-LLM）甚至把二者拆到不同 GPU 池上做 **PD 分离**（Prefill/Decode disaggregation）。

### 为什么输出是流式且逐 Token 生成的

- **自回归本质**：主流 LLM 是自回归语言模型，第 N+1 个 Token 的概率分布依赖于前 N 个 Token。下一个词必须等上一个词确定后才能计算，这在数学上就决定了无法一次性并行生成全文。
- **因此可以流式返回**：既然 Token 本来就是一个接一个产生的，服务端每算出一个 Token 就能立刻推送给客户端（SSE / WebSocket），无需等整段生成完。这显著降低用户的"感知延迟"，体验上像打字机。
- **工程价值**：流式让首字节更快到达、便于中途取消（节省算力）、便于前端边收边渲染。注意流式只是"把已产出的 Token 尽早吐出"，并不改变总生成时间，也不加快生成速度。

### 推理成本由什么决定

成本主要由"占用多少 GPU、占用多久"决定，可拆解为以下因素：

- **模型规模与精度**：参数量越大、精度越高（FP16 > FP8 > INT4），单 Token 计算量和显存占用越大。量化是最直接的降本手段。
- **输入/输出 Token 数**：Prefill 成本随输入长度增长；Decode 成本随输出长度增长。多数 API 按 input/output token 分别计费，且 output 通常更贵（因为 Decode 低效）。
- **KV Cache 显存**：显存占用 ≈ 层数 × 2 × 隐藏维 × 上下文长度 × batch × 精度字节数。长上下文和高并发会迅速吃满显存，限制 batch 大小，进而拉高单请求均摊成本。
- **batch 与 GPU 利用率**：Decode 是访存瓶颈，单请求时 GPU 算力大量闲置；通过 continuous batching 把多请求拼批能大幅摊薄成本。利用率是降本核心。
- **硬件与优化手段**：GPU 型号、FlashAttention、PagedAttention、投机解码、量化等共同决定实际吞吐（tokens/s/GPU），而吞吐直接倒数成单位 Token 成本。

## 核心内容

- **两阶段计算**：推理 = Prefill（并行、算力瓶颈）+ Decode（串行、带宽瓶颈），两者优化思路完全不同。
- **KV Cache**：缓存历史 Token 的 Key/Value，避免每步重算，用显存换算力；是长上下文和高并发下的头号显存消耗者。
- **自回归 + 采样**：逐 Token 生成，每步经 softmax 得到分布，再用 temperature/top-p/top-k 等策略采样，控制随机性与多样性。
- **关键指标**：TTFT（首 Token 延迟）、TPOT/ITL（Token 间延迟）、Throughput（吞吐 tokens/s）、并发数、单 Token 成本。
- **核心优化技术**：Continuous Batching（动态拼批）、PagedAttention（KV Cache 分页管理，vLLM 核心）、FlashAttention（IO 感知的注意力）、量化（INT8/INT4/FP8）、投机解码（Speculative Decoding）、PD 分离、Prefix Cache（公共前缀复用）。
- **典型框架**：vLLM、TensorRT-LLM、SGLang、TGI、llama.cpp（本地/CPU）。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | 模型训练 / 权重、Transformer 架构、KV Cache | 推理消费训练产出的权重；Transformer 的注意力结构决定了 Prefill/Decode 的计算特征与 KV Cache 的存在 |
| 下游 | 模型服务化（Serving）、Agent/RAG 应用、API 计费 | 推理是上层应用的运行时底座；Agent 的多轮调用、RAG 的长上下文都直接放大推理成本与延迟 |
| 易混淆 | 训练 vs 推理、流式 vs 并行生成、吞吐 vs 延迟 | 训练反向传播更新权重、推理只做前向；流式是"尽早吐出"而非并行生成；高吞吐（大 batch）常以牺牲单请求延迟为代价，二者需权衡 |

## 典型工程场景

- 在线 Chat/Copilot 服务：高并发、要求低 TTFT 和流式输出。
- AI 编码助手的代码补全：对延迟极度敏感，常用小模型 + 投机解码 + Prefix Cache。
- RAG 问答：prompt 很长（检索片段），Prefill 成本高，需要长上下文优化与 Prefix Cache。
- Agent 多步推理/工具调用：一次任务多次模型调用，累积 Token 成本与延迟是主要瓶颈。
- 批量离线处理（文档摘要、数据标注、评测）：追求最大吞吐而非低延迟，用大 batch 把 GPU 打满。
- 端侧/本地部署：用 llama.cpp + 量化在消费级硬件上跑小模型。

## 适用边界

**适合使用**

- 需要把训练好的模型对外提供实时或批量服务的任何场景。
- 对延迟、吞吐、成本有明确工程指标、需要专门调优的生产部署。
- 高并发、长上下文、多租户等需要精细管理 GPU 显存与批处理的场景。

**不适合使用**

- 还在改模型本身（训练/微调）的阶段——那是训练而非推理工程。
- 极低频、无延迟要求的一次性调用，直接用托管 API 即可，自建推理栈反而增加复杂度和成本。
- 任务其实不需要大模型（能用规则、检索、小分类器解决）时，过度投入推理优化是方向性错误。

## 常见误区

- **误以为流式比非流式"更快生成"**：流式只是尽早吐出已产出的 Token，总生成时间不变。
- **把延迟和吞吐当同一件事**：增大 batch 提高吞吐，往往会拉高单请求延迟，二者需按场景权衡。
- **忽视 KV Cache 显存**：只按模型权重估显存，上线后被长上下文 × 高并发的 KV Cache 撑爆 OOM。
- **以为量化一定掉点严重**：现代 INT8/FP8/INT4 量化在多数任务上精度损失很小，是性价比极高的降本手段。
- **用单请求 benchmark 代表线上性能**：单请求下 GPU 严重闲置，真实吞吐要在 continuous batching + 并发压测下测。
- **把 Prefill 和 Decode 当成同质计算一刀切优化**：二者瓶颈不同（算力 vs 带宽），需要分别对待。

## 我的结论

> 推理是模型"从能跑通到能上线"的分水岭：只要模型要真实对外服务，就必然进入推理工程的范畴。该不该投入专门的推理优化，取决于规模与约束——当面临高并发、长上下文、低延迟或明确的成本压力时，应认真选型推理框架（vLLM/TensorRT-LLM/SGLang）并围绕 KV Cache、batching、量化做调优，这里的收益（成本降一个数量级、延迟减半）非常可观。反之，若调用低频、无硬性 SLA，或任务根本不需要大模型，直接用托管 API 甚至不用大模型，自建推理栈只会徒增复杂度。判断的核心抓手永远是三个数：TTFT、吞吐（tokens/s/GPU）、单 Token 成本。

## 待解决问题

- [ ] 投机解码（Speculative Decoding）在不同任务/模型组合下的实际加速比与草稿模型选型的最佳实践？
- [ ] PD 分离（Prefill/Decode disaggregation）在哪些流量特征下才真正划算，拆分带来的网络/KV 传输开销如何权衡？
- [ ] 不同量化方案（GPTQ / AWQ / FP8 / INT4）在具体业务任务上的精度-吞吐权衡的量化对比数据？

## 参考资料

- Vaswani et al., "Attention Is All You Need"（Transformer 原始论文）
- Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention"（vLLM 论文，SOSP 2023）
- Dao et al., "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"
- Leviathan et al., "Fast Inference from Transformers via Speculative Decoding"（投机解码）
- vLLM 官方文档（docs.vllm.ai）
- NVIDIA TensorRT-LLM 官方文档
- Pope et al., "Efficiently Scaling Transformer Inference"（Google，推理扩展）
