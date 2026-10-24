# Quantization / 量化

> **状态**：🟢 已理解 ｜ **深度**：D 暂不投入 ｜ **更新**：2026-09-15
> **上级**：[90-模型原理](README.md) ｜ **依赖**：Inference

## 一句话定义

> 量化是把神经网络的权重和/或激活值从高精度浮点数（如 FP32/FP16）映射为更低位宽的表示（如 INT8、INT4、FP8），用更少的比特数存储和计算，从而压缩模型体积、降低显存占用并加速推理。

## 解决的问题

> 大模型以 FP16/FP32 存储时，参数体量直接决定显存门槛：一个 7B 模型 FP16 就要约 14GB 显存，70B 需要上百 GB，单卡乃至单机都难以装下。推理时每生成一个 token 都要把全部权重从显存搬到计算单元，显存带宽成为瓶颈，吞吐低、延迟高、单位 token 成本昂贵。量化出现之前，想在消费级显卡或边缘设备上跑大模型几乎不可能，只能堆更贵的硬件或放弃本地部署。


## 待回答的问题

- [x] 量化如何降低部署成本
- [x] 量化对效果的影响
- [x] 什么场景下值得用量化模型

### 量化如何降低部署成本

- 显存/内存直降：位宽从 16-bit 降到 8-bit 约省一半，降到 4-bit 约省到 1/4。7B 模型 FP16 约 14GB，INT4 后约 4GB 左右，可在单张消费级 GPU（甚至部分笔记本）上运行。
- 带宽即速度：LLM 自回归解码是显存带宽受限（memory-bound）的，权重读取量减少，直接提升 decode 阶段吞吐，降低每 token 延迟。
- 吞吐与并发：显存省下来可容纳更大的 batch 和更长的 KV cache，提升单卡并发用户数，摊薄单位请求成本。
- 硬件门槛下移：原本需要多卡/高端卡的模型可降级到更便宜的卡，减少整机和电力成本。
- 需要注意：低位宽并非总能线性提速——若 kernel 需要反量化回 FP16 做矩阵乘（如朴素 weight-only 实现）、或算子未针对该位宽优化，收益会打折；真正的加速依赖硬件原生低精度算力（INT8 Tensor Core、FP8 等）和高效 kernel。

### 量化对效果的影响

- 总体规律：位宽越低精度损失越大。INT8（尤其 weight-only）通常几乎无损；INT4 在好的算法下困惑度/下游指标损失很小（常见 1% 内到个位数百分比）；更低（INT3/INT2）或不加校准则明显退化。
- 权重 vs 激活：权重分布较规整、相对好量化；激活含大幅度 outlier（离群值），直接量化误差大，这是 LLM 量化的主要难点（SmoothQuant、AWQ 等正是处理 outlier）。
- 粒度影响：per-tensor 最省但最糙；per-channel / group-wise（如每 128 个权重一组一个 scale）精度更好，是 4-bit 的常见做法。
- PTQ vs QAT：训练后量化（PTQ）成本低、可能有损；量化感知训练（QAT）通过训练补偿误差、精度更高但需重训，成本大。
- 任务敏感性：对长链推理、数学、代码等高精度任务，低位量化的退化更容易被放大；对一般对话/检索类任务影响较小。需用贴近实际的评测集验证，而非只看 perplexity。

### 什么场景下值得用量化模型

| 场景 | 是否值得 | 说明 |
| --- | --- | --- |
| 显存/内存受限（消费级 GPU、边缘、本地部署） | 很值得 | 往往是能否跑起来的前提，INT4 让大模型落地单卡 |
| 推理成本敏感、高并发在线服务 | 值得 | 省显存换更大 batch 与并发，摊薄单位成本 |
| 延迟敏感、decode 带宽受限 | 值得 | 权重读取减少直接降延迟 |
| 对精度要求极高的任务（金融、医疗、复杂推理） | 谨慎 | 需严格评测，必要时用 INT8 或 QAT，或不量化 |
| 训练/微调阶段 | 一般不用（除 QLoRA 等） | 量化主要服务推理；QLoRA 用 4-bit 冻结权重 + LoRA 微调是特例 |
| 硬件不支持目标低精度算力 | 收益有限 | 可能省显存但不提速，需评估 |

## 核心内容

- 基本映射：`q = round(x / scale) + zero_point`，`scale` 决定量化步长，`zero_point` 处理非对称分布；反量化 `x ≈ (q - zero_point) * scale`。
- 对称 vs 非对称：对称量化 zero_point=0，适合以 0 为中心的权重；非对称更贴合激活等偏移分布。
- 粒度：per-tensor（整张量一个 scale）→ per-channel/per-token → group-wise（每组若干元素一个 scale），粒度越细精度越好、开销越大。
- 量化对象：weight-only（只量权重，激活仍 FP16，解码带宽收益大，典型 INT4）；weight+activation（W8A8 等，可用整数算力进一步提速，需处理激活 outlier）；KV cache 量化（省长上下文显存）。
- PTQ（训练后量化）：无需重训，用少量校准数据统计分布，代表方法 GPTQ、AWQ、SmoothQuant。
- QAT（量化感知训练）：训练中插入伪量化节点，用直通估计器（STE）让梯度回传，精度高但成本高。
- Outlier 处理：激活离群值是 LLM 量化核心难点，SmoothQuant 把量化难度从激活迁移到权重，AWQ 按激活重要性保护关键权重通道。
- 常见格式/生态：GGUF（llama.cpp，Q4_K_M 等）、GPTQ、AWQ、bitsandbytes（NF4/QLoRA）、FP8（H100/新硬件原生支持）。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | 浮点数值表示与推理（Inference）、Transformer 权重结构 | 量化建立在模型已有 FP16/FP32 权重与推理流程之上，是推理优化的一环 |
| 下游 | 本地/边缘部署、推理服务（vLLM/llama.cpp/TensorRT-LLM）、QLoRA 微调 | 量化后的权重被这些运行时与微调方案直接消费 |
| 易混淆 | 剪枝（Pruning）、蒸馏（Distillation） | 三者都是模型压缩：量化降位宽、剪枝去冗余连接/结构、蒸馏用小模型学大模型；可组合使用但机制不同 |

## 典型工程场景

- 消费级 GPU 或 Mac 上本地跑 LLM（llama.cpp + GGUF Q4），无需高端卡。
- 在线推理服务用 AWQ/GPTQ INT4 权重 + vLLM，提升单卡并发与吞吐、降成本。
- 边缘/端侧设备（手机、嵌入式、IoT）部署小模型，受限于内存与功耗。
- 长上下文服务通过 KV cache 量化（INT8/FP8）容纳更长序列。
- 资源有限下微调：QLoRA 以 4-bit NF4 冻结底座 + LoRA 适配器，在单卡微调大模型。
- H100 等新硬件用 FP8 做训练/推理，兼顾精度与原生算力加速。

## 适用边界

**适合使用**

- 显存/内存是部署硬约束，必须压缩才能装下模型。
- 推理成本或延迟敏感、需要高并发的在线服务。
- 目标硬件具备低精度原生算力（INT8/FP8 Tensor Core）或有高效量化 kernel。
- 任务对精度容忍度较高（一般对话、检索、摘要等）。

**不适合使用**

- 对数值精度极敏感的高风险任务（复杂数学/推理、金融结算）且未经充分评测。
- 硬件/运行时不支持目标位宽，量化只增复杂度不带来提速。
- 模型已很小、显存充裕，量化收益不足以抵消精度损失与工程成本。
- 需要继续全量训练的权重（训练一般用更高精度，量化主要服务推理）。

## 常见误区

- 以为量化一定提速：若需反量化回 FP16 计算或缺高效 kernel，可能只省显存不提速。
- 把位宽越低当越好：INT3/INT2 常严重掉点，需权衡而非一味压缩。
- 只看 perplexity 判断无损：困惑度变化小不代表下游任务（代码、推理）不退化，应用贴近业务的评测集。
- 混淆 weight-only 与 W8A8：前者激活仍是 FP16，加速主要来自带宽；后者才用整数算力算矩阵乘。
- 忽视激活 outlier：对激活直接做朴素量化误差巨大，这是 LLM 量化失败的常见原因。
- 认为量化=永久修改原模型：量化产物是另一套权重，原始高精度权重仍可保留。

## 我的结论

> 当"能不能跑起来"或"单位成本"成为瓶颈时，量化几乎是首选且性价比最高的手段——尤其是显存受限的本地/边缘部署和高并发在线推理，INT4 weight-only（AWQ/GPTQ/GGUF）通常以极小精度代价换来数倍的显存与成本收益。
> 反过来，当显存充裕、任务对精度极度敏感、或目标硬件没有对应低精度算力时，不该盲目量化：此时要么保持 FP16，要么退一步用 INT8，并且无论哪种都必须用贴近真实业务的评测集验证掉点是否可接受。一句话：量化是"用可控的精度换显存和速度"的工程取舍，决策依据是硬件约束 + 任务精度容忍度，而非默认开或默认关。

## 待解决问题

- [ ] 对复杂推理/代码任务，INT4 相比 INT8 的实际掉点在当前主流算法下有多大，是否有稳定可复用的评测基线？
- [ ] FP8 与 INT8/INT4 在新硬件（H100/新一代 GPU）上的精度-速度权衡边界具体如何，各自最佳适用位置是什么？
- [ ] KV cache 量化在超长上下文下对注意力质量的实际影响程度，以及与权重量化叠加时的误差累积规律。

## 参考资料

- GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers（Frantar et al., 2022）
- AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration（Lin et al., 2023）
- SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models（Xiao et al., 2022）
- LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale（Dettmers et al., 2022）
- QLoRA: Efficient Finetuning of Quantized LLMs（Dettmers et al., 2023）
- Quantizing deep convolutional networks for efficient inference: A whitepaper（Google, 2018）
- NVIDIA TensorRT 与 Transformer Engine（FP8）官方文档
- llama.cpp / GGUF 量化格式文档；Hugging Face Optimum 与 bitsandbytes 官方文档
