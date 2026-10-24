# 03 Agent 核心

> 主线域 ｜ 目标：理解 Agent 的本质，不把 Agent 简单理解成「调用 LLM 的程序」。

## 本域定位

本域回答的是**Agent 的组成与运行机制**：它由什么构成，怎么转起来，转起来之后需要什么。

**本域最关键的缺口是 `Agent Loop`（🔴）**。它是整个 Agent 体系的地基——
不真正理解循环和状态，后面的 Planning / Memory / Multi-Agent 都只能停留在名词层面。

**结构决策**：原地图从「Agent 是什么」直接跳到「Loop / Planning / Memory」这些组成部件，
中间**缺少一层**——业界沉淀的、可直接套用的**组织模式**（ReAct、Plan-and-Execute、
Orchestrator-Worker 等）。这一层已独立为 [04-Agent设计模式](../04-Agent设计模式/README.md)，
它是从「理解 Agent」到「设计 Agent」的桥梁，请不要跳过。

## 知识点清单

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| [Agent 是什么](01-Agent是什么.md) | 🟢 | A | 理解 Agent 的本质与边界 |
| [Agent Loop](02-Agent-Loop.md) | 🔴 | A | Agent 的心脏，当前最关键的红色缺口 |
| [Planning / 规划](03-Planning.md) | 🟢 | A | 把大目标拆成可执行步骤 |
| [Reasoning / Decision](04-Reasoning与Decision.md) | 🟢 | A | Agent 的决策从何而来 |
| [Memory / 记忆](05-Memory.md) | 🟢 | A | 跨会话保持信息的能力 |
| [Reflection / 反思](06-Reflection.md) | 🟢 | B | 让 Agent 检查并修正自己的输出 |
| [Multi-Agent / 多智能体](07-Multi-Agent.md) | 🟢 | B | 多个 Agent 协作的收益与代价 |

## 待建文件（已识别的知识缺口）

这些知识点**已登记但尚未建文件**，学到时再创建，避免产生大量空壳。

| 知识点 | 优先级 | 为什么需要 |
| --- | --- | --- |
| Agent 组织模式 | **高（必修）** | 已独立为 04-Agent设计模式 域 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 02-AI应用开发 | 前置能力 |
| 04-Agent设计模式 | 本域的直接延续，从组成到组织 |
| 07-Workflow编排 | 边界需要明确划分 |
| 08-Agent工程化 | 本域概念在生产环境中的加固 |

## 建议学习顺序

1. **Agent 是什么** — 先划清与普通 LLM 应用的边界
2. **Agent Loop**（🔴 优先攻克）— 心脏
3. **Reasoning / Decision** — 循环里每步怎么决策
4. **Memory** — 循环之外的信息保持
5. **Planning** — 决策的高级形态
6. **Reflection** — 自我修正
7. **Multi-Agent** — 最后再看，避免过早复杂化

> 学完本域后，请转向 **04-Agent设计模式** —— 那里才是"怎么把 Agent 搭出来"。

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 建立本域骨架 |
| 2026-09-15 | 完成本域知识点笔记，状态转为 🟢 |
