# Structured Output / 结构化输出

> **状态**：🟢 已掌握 ｜ **深度**：A 深入掌握 ｜ **更新**：2026-09-15
> **上级**：[02-AI应用开发](README.md) ｜ **依赖**：Prompt Engineering

## 一句话定义

> Structured Output 是让 LLM 按预定义的结构（通常是符合某个 JSON Schema 的 JSON）输出，从而让它的结果能被程序稳定、可靠地解析和消费，而不是一段自由文本。

## 解决的问题

> LLM 天然输出自由文本，但软件系统需要的是**确定结构的数据**（字段、类型、枚举）。
> 如果只靠提示"请返回 JSON",模型经常加解释文字、用 markdown 包裹代码块、字段名漂移、少逗号/多逗号、类型不对……导致程序解析失败。这让"把模型接进系统"变得脆弱、充满 try/except。结构化输出把"输出符合约定结构"从碰运气变成可保证，是 LLM 作为**软件组件**的关键一步。


## 待回答的问题

- [x] 为什么需要结构化输出
- [x] JSON / Schema 的定义方式
- [x] 如何让模型输出可靠地进入程序处理
- [x] Schema 校验失败时的处理策略

### 为什么需要结构化输出

因为程序消费的是数据不是散文。结构化输出让模型结果可以**直接反序列化成对象**、落库、传给下游 API、驱动 UI。它带来：确定的字段与类型、可被 schema 校验、消除解析脆弱性、让 LLM 真正成为可组合的系统组件。没有它，每次调用都要写易碎的文本解析代码。它也是 Tool Calling 的基础——工具参数本质就是一个结构化对象。

### JSON / Schema 的定义方式

- **JSON** 是事实标准的数据交换格式（键值对、数组、基本类型）。
- **JSON Schema** 用来声明"合法的 JSON 长什么样":字段名、类型（string/number/boolean/array/object）、是否必填（required）、枚举（enum）、嵌套结构、约束（min/max/pattern）等。
- 实际工程里常用**代码定义 schema** 再自动生成：
  - Python：Pydantic 模型 / dataclass → JSON Schema。
  - TS：Zod → JSON Schema。
- 现代 API 直接接收 schema：OpenAI 的 `response_format: json_schema`（strict 模式）、function/tool 的 `parameters` 字段、Anthropic 的 tool input schema 等。

### 如何让模型输出可靠地进入程序处理

可靠性从低到高：
1. **提示要求 JSON**：最弱，必须配容错解析。
2. **JSON mode**：平台保证输出是**合法 JSON**（但不保证符合你的 schema）。
3. **Schema-constrained / Structured Outputs（strict）**：平台在解码阶段用约束解码（constrained decoding / grammar）保证输出**严格符合给定 schema**——最可靠，字段和类型都对。
4. **Function/Tool Calling**：模型按工具的参数 schema 产出结构化参数，本质也是结构化输出。
- 工程铁律：即便用了 strict 模式，代码侧仍要做一次 **schema 校验 + 反序列化**，把"信任模型"降级为"校验后信任"。

### Schema 校验失败时的处理策略

- **重试（retry）**：把校验错误信息回灌给模型，要求修正后重出（self-healing / reask）。
- **约束解码优先**：能用 strict/grammar 模式就用，从源头避免失败。
- **容错解析**：剥掉 markdown 代码块围栏、提取第一个 JSON 对象、修复常见小错（尾逗号）。
- **降级（fallback）**：多次失败后走默认值、人工兜底或返回明确错误，而不是让坏数据流入系统。
- **限流与上限**：设置最大重试次数，避免无限循环和成本失控。
- **可观测**：记录失败样本用于改进 prompt/schema。

## 核心内容

- **目标**：让输出"可被程序确定性消费",而非自由文本。
- **三层保障**：平台约束（json_schema/json mode/tool）→ 代码校验（Pydantic/Zod）→ 失败处理（重试/降级）。
- **约束解码**：strict 模式通过限制每步可生成的 token，使输出一定符合语法/schema，这是可靠性的根本来源。
- **schema 即契约**：schema 同时作为"给模型的格式说明"和"给程序的校验规则",单一事实源。
- **代价**：strict/复杂 schema 可能略增延迟、限制模型表达；过度嵌套的 schema 成功率下降，宜扁平、字段清晰、带描述。
- **与 Tool Calling 同源**：工具参数就是结构化输出的一个应用。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | Prompt Engineering、JSON Schema、约束解码 | 结构化输出建立在提示与 schema 约束之上 |
| 下游 | Tool Calling、Agent、数据管道 | 工具参数/系统集成都依赖结构化输出 |
| 易混淆 | "提示里要求 JSON"、自由文本解析 | 单纯提示不保证合规；strict 模式才有保证 |

## 典型工程场景

> - 信息抽取：把非结构化文本转成字段化 JSON 入库。
> - 分类/打标：输出固定枚举值。
> - 表单/配置生成：按 schema 生成可直接使用的配置。
> - 作为 Tool Calling 的参数生成。
> - 驱动 UI：返回结构化数据渲染卡片/表格。

## 适用边界

**适合使用**

- 输出要被程序解析、落库、传给下游时
- 需要稳定字段、类型、枚举的任务
- 工具调用参数生成

**不适合使用**

- 面向人阅读的自由文本（文章、对话、创意写作）
- schema 极度复杂/深嵌套导致成功率和表达力下降时（应拆分或简化）
- 需要模型自由推理叙述的中间过程（可另开字段，别硬塞进结构）

## 常见误区

- 以为"提示里写了返回 JSON"就可靠——不加约束解码仍常失败。
- 用了 strict 模式就不在代码侧校验——仍应校验后再信任。
- schema 设计过度嵌套/字段含糊——降低成功率，应扁平+带 description。
- 没有失败处理策略——校验失败直接崩，缺重试/降级。
- 把需要人读的长文也硬塞进 JSON 字段——牺牲质量。

## 我的结论

> 结构化输出是让 LLM 从"聊天模型"变成"系统组件"的关键接口：它把"输出能被程序吃掉"从碰运气变成有保证。
> - **该用它**：任何输出要进程序、落库、传下游、做工具参数的场景——默认就该上。
> - **怎么用才稳**：优先平台的 strict/json_schema 约束解码，schema 用 Pydantic/Zod 定义做单一契约，代码侧永远再校验一次，并备好重试/降级。
> - **不该用**：给人读的自由文本、过度复杂的嵌套结构。
> 心智模型：schema 是模型与程序之间的"合同",约束解码保证不违约，代码校验是"签收前再验货"。

## 待解决问题

- [ ] strict/约束解码对延迟和输出质量的实际影响有多大
- [ ] 复杂嵌套 schema 的拆分与成功率优化策略
- [ ] 跨厂商（OpenAI/Anthropic/开源）结构化输出能力与可移植性对比

## 参考资料

- OpenAI Structured Outputs / JSON mode 文档
- JSON Schema 规范（json-schema.org）
- Pydantic 文档；Zod 文档
- Anthropic Tool use（input schema）文档
