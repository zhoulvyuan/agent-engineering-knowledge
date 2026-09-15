# 09 AI Coding

> 主线域 + 贯穿线 ｜ 目标：理解 AI 如何进入软件研发过程，以及 AI Coding 工具背后的 Agent 工程思想。

## 本域定位

本域有**双重身份**，这是原地图没有体现的：

1. **作为学习内容**：Coding Agent 是目前最成熟、最复杂的 Agent 应用，
   研究它等于研究 Agent 工程的最佳实践。
2. **作为学习手段**：你日常使用的 DSH 等工具，本身就是**实践环境**。
   不要把 AI Coding 排到学习路线的最后——它是贯穿全程的加速器。

**视角提示**：本域与 [03-Agent核心](../03-Agent核心/README.md)、
[05-工具与扩展](../05-工具与扩展/README.md) 存在内容重叠。
本知识库的划分是：

- **03 / 05 域**：讲**通用原理**（Loop、Tool、MCP、Skill 是什么）
- **本域**：讲**具体实现**（Claude Code / OpenCode / DSH 是怎么做的）

看本域时，请始终问："它这样设计，对应的是哪个通用原理？"——这是把工具使用
转化为工程能力的关键动作。

## 知识点清单

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| [AI Coding 基本模式](01-AI-Coding基本模式.md) | 🟡 | B | AI 进入研发过程的三种形态 |
| [Claude Code](02-Claude-Code.md) | 🟡 | B | 生产级 Coding Agent 的参考实现 |
| [OpenCode](03-OpenCode.md) | 🟡 | B | 开源 Coding Agent 架构参考 |
| [DeepSeek Harness](04-DeepSeek-Harness.md) | 🟡 | B | DSH 的架构与扩展机制 |
| [AI Coding Workflow](05-AI-Coding-Workflow.md) | 🟡 | B | 把 AI 嵌入完整研发流程 |
| [Agentic Software Engineering](06-Agentic-Software-Engineering.md) | ⚪ | B | 研发流程的长期演进方向 |

## 待建文件（已识别的知识缺口）

这些知识点**已登记但尚未建文件**，学到时再创建，避免产生大量空壳。

| 知识点 | 优先级 | 为什么需要 |
| --- | --- | --- |
| Coding Agent 的上下文策略 | 中 | 大仓库下的代码检索与上下文选择 |
| AI 生成代码的质量与安全审查 | 中 | 生成代码的漏洞与许可证风险 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 03-Agent核心 | Coding Agent 是 Loop 的最佳实例 |
| 05-工具与扩展 | Tools / Skills / MCP 的落地样本 |
| 08-Agent工程化 | Coding Agent 是工程化的集中体现 |

## 建议学习顺序

1. **AI Coding 基本模式** — 先建立整体分类
2. **Claude Code** — 最成熟的参考实现，建议重点研究
3. **DeepSeek Harness** — 你日常使用的工具，理解其架构
4. **OpenCode** — 开源视角，看模型适配与可扩展性
5. **AI Coding Workflow** — 把工具用成流程
6. **Agentic Software Engineering** — 趋势与边界

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 建立本域骨架 |
