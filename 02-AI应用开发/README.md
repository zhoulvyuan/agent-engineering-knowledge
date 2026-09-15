# 02 AI 应用开发

> 主线域 ｜ 目标：理解如何把 LLM 从「聊天模型」变成可以进入软件系统的能力。

## 本域定位

本域是**从概念走向工程的第一站**。核心问题只有一个：

> 怎么让一个只会输出文本的模型，变成软件系统里一个**可靠的、可被程序调用的组件**？

这条线索的推进顺序是：控制说什么（Prompt）→ 控制输出形状（Structured Output）
→ 让它能行动（Tool Calling）→ 管理它的全部输入（Context Engineering）。

## 知识点清单

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| [Prompt Engineering](01-Prompt-Engineering.md) | 🟢 | A | 用自然语言控制模型行为的基础手段 |
| [Structured Output / 结构化输出](02-Structured-Output.md) | 🟡 | A | 让模型输出能被程序稳定消费 |
| [Tool Calling / 工具调用](03-Tool-Calling.md) | 🟡 | A | 模型从「说」到「做」的第一步 |
| [Context Engineering / 上下文工程](04-Context-Engineering.md) | 🟡 | A | 把上下文当作可设计的工程对象 |

## 待建文件（已识别的知识缺口）

这些知识点**已登记但尚未建文件**，学到时再创建，避免产生大量空壳。

| 知识点 | 优先级 | 为什么需要 |
| --- | --- | --- |
| LLM API 工程实践 | **高（必修）** | 消息角色结构、streaming、temperature/top_p/stop/seed、prompt caching、rate limit、错误重试。从概念到代码的落地层，目前完全缺失 |
| Streaming 与交互体验 | 中 | 流式输出、用户打断、部分结果渲染 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 01-基础认知 | 提供概念基础 |
| 03-Agent核心 | 本域能力是 Agent 的前置条件 |
| 06-知识检索RAG | RAG 本质是被注入 Context 的外部知识 |
| 08-Agent工程化 | 本域的设计需要在工程化阶段被验证和加固 |

## 建议学习顺序

1. **Prompt Engineering**（已掌握，作为参考随时回看）
2. **Structured Output** — 输出进入程序的入口
3. **Tool Calling** — 从「说」到「做」，Agent 的前置条件
4. **Context Engineering** — 把前面三者统一到"上下文"这个工程对象下

> **缺失环节**：本域目前**跳过了一个落地层**——LLM API 本身的工程实践
> （消息角色、streaming、参数、缓存、限流、重试）。它是 Prompt 和 Tool Calling
> 之间的桥梁，已登记为缺口，见下表。

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 建立本域骨架 |
