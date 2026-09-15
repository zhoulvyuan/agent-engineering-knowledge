# 04 Agent 设计模式

> 主线域 ｜ **本域为已识别的关键缺口，暂未建独立文件。**

## 为什么需要这个域

原知识地图从「Agent 是什么」直接跳到「Agent Loop / Planning / Memory」这些**组成部件**，
中间缺失了一层：**业界沉淀的、可直接套用的组织模式**。

这一层的重要性在于：

- 它**不是概念，而是架构模板**。拿到一个业务需求时，你要判断的是
  "这个场景该用 Router 还是 Orchestrator"，而不是"我要写一个 while 循环"。
- 它是从**理解**到**设计**的唯一桥梁。没有它，Agent 设计会退化成"凭感觉拼装"。
- 它是 [08-Agent工程化](../08-Agent工程化/README.md) 和 [11-架构设计](../11-架构设计/README.md) 的**共同前置**。

## 知识点清单（待建文件）

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| ReAct | ⚪ | A | 推理与行动交替：Reason → Act → Observe |
| Plan-and-Execute | ⚪ | A | 先出完整计划，再逐步执行 |
| Reflexion | ⚪ | B | 失败后自我反思并重试 |
| Router / 分发 | ⚪ | A | 先分类，再交给最合适的处理者 |
| Orchestrator-Worker | ⚪ | A | 一个调度者拆解并分发给多个执行者 |
| Evaluator-Optimizer | ⚪ | A | 生成者与评估者对抗迭代 |
| Handoff / Sub-agent | ⚪ | A | 任务与上下文在 Agent 间转移 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 03-Agent核心 | 本域是其直接延续：从「组成」到「组织」 |
| 07-Workflow编排 | 设计模式是 Workflow 与 Agent 的结合点 |
| 08-Agent工程化 | 模式决定评估与可观测的粒度 |
| 11-架构设计 | 模式是架构设计的基本积木 |

## 建议学习顺序

1. 先吃透 **ReAct** —— 它是 Agent Loop 的最经典实现，理解了它就理解了 Loop
2. 再学 **Router** 和 **Orchestrator-Worker** —— 覆盖大多数实际场景
3. **Plan-and-Execute** / **Evaluator-Optimizer** —— 面向复杂任务
4. **Reflexion** / **Handoff** —— 按需补充

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 识别本域为关键缺口，建立占位 |
