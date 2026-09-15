# AI 工程能力体系 V1

> ⚠️ **本文件已归档，不再是知识库的入口。**
>
> 内容已全部迁移并重组到新的多文件知识库中，请从 **[知识地图.md](../知识地图.md)** 开始阅读。
>
> 保留本文件的目的是留存 V1 的原始表述与结构，供对照参考。
> **日常维护请修改新结构，不要修改本文件**，否则会产生两份冲突的事实来源。

---

> 本文是我的 AI 工程能力知识地图。
>
> **核心目标：** 以已有的 Java 软件工程能力为基础，从 AI 使用与应用开始，逐步深入到模型、Agent 和 AI 工程化，最终形成能够理解、设计、开发和评估 AI 应用/Agent 系统的工程能力。
>
> **学习原则：** 不走“数学 → 机器学习 → 深度学习 → Transformer”的学院派路线；优先从使用和工程问题出发，在真正需要时再补充原理。学习重点不是追逐零散技术，而是理解技术解决的问题、所在层次、相互关系、适用边界和工程价值。

---

## 0. 使用说明

### 0.1 这张地图是什么

这不是 AI 技术名词清单，也不是详细学习笔记。

它负责回答：

- AI 工程有哪些核心能力？
- 各能力之间是什么关系？
- 我现在处于什么水平？
- 下一步应该学习什么？
- 哪些技术需要深入，哪些只需要理解概念？

### 0.2 学习循环

```text
知识地图
  ↓
发现知识缺口
  ↓
项目内开一个专题会话
  ↓
理解 / 对比 / 实践
  ↓
形成结论
  ↓
更新知识地图
  ↓
继续发现新的知识缺口
```

### 0.3 知识状态

- ⚪ 未开始
- 🔴 不理解
- 🟡 理解中
- 🟢 已掌握
- 🔵 已实践

> V1 阶段不要急于给所有节点标记“已掌握”。状态的意义是帮助识别知识缺口。

---

# 1. AI 基础认知

> 目标：理解现代大模型和 AI 应用运行的基本机制。不追求数学推导，先建立工程认知。

## 1.1 LLM / 大语言模型
状态：🟡

需要理解：

- LLM 本质上是什么
- 训练与推理的基本区别
- 模型能力从哪里来
- 模型为什么会产生幻觉
- 模型能力与上下文之间的关系

## 1.2 Token
状态：🟡

需要理解：

- Token 是什么
- Token 与文本的关系
- Token 与上下文窗口的关系
- Token 与成本、性能的关系

## 1.3 Context / 上下文
状态：🟡

需要理解：

- Context 是什么
- 上下文窗口是什么
- 为什么上下文管理成为 Agent 的核心问题
- Context 与 Memory 的区别

## 1.4 Transformer
状态：⚪

目标：

- 理解 Transformer 在 LLM 中解决什么问题
- 理解 Attention 的基本思想
- 暂不要求深入数学推导

## 1.5 Embedding
状态：🟡

需要理解：

- Embedding 是什么
- 为什么可以把文本转换成向量
- Embedding 与语义检索、RAG 的关系

---

# 2. AI Application

> 目标：理解如何把 LLM 从“聊天模型”变成可以进入软件系统的能力。

## 2.1 Prompt Engineering
状态：🟢

重点：

- 指令设计
- 角色与约束
- Few-shot
- 输出格式控制

## 2.2 Structured Output
状态：🟡

重点：

- 为什么需要结构化输出
- JSON / Schema
- 如何让模型输出进入程序处理

## 2.3 Tool Calling
状态：🟡

重点：

- Tool Calling 解决什么问题
- 模型如何决定调用工具
- 工具定义、参数、结果
- Tool Calling 与传统 API 调用的区别
- Tool Calling 是 Agent 的基础能力之一

## 2.4 Context Engineering
状态：🟡

重点：

- Context 为什么成为新的工程对象
- 如何组织指令、工具、知识、历史和状态
- Context 与 Prompt Engineering 的关系
- Context 与 Agent 的关系

## 2.5 AI Application Architecture
状态：⚪

需要逐步理解：

```text
用户
 ↓
AI Application
 ↓
LLM
 ↓
Context / Prompt
 ↓
Tools / Knowledge
 ↓
业务系统
```

---

# 3. Agent

> 目标：理解 Agent 的本质，不把 Agent 简单理解成“调用 LLM 的程序”。

## 3.1 Agent 是什么
状态：🟡

需要回答：

- Agent 与普通 LLM Application 的区别
- Agent 的自主性来自哪里
- Agent 的基本组成

## 3.2 Agent Loop
状态：🔴

重点：

```text
Observe
  ↓
Reason / Decide
  ↓
Act
  ↓
Observe
  ↓
...
  ↓
Terminate
```

需要理解：

- Loop 为什么是 Agent 的核心
- State 在 Loop 中的作用
- Tool Calling 如何进入 Loop

## 3.3 Planning
状态：⚪

需要理解：

- Agent 为什么需要 Planning
- Planning 与普通 Workflow 的区别
- 动态规划与预定义流程

## 3.4 Reasoning / Decision
状态：🟡

需要理解：

- 模型推理在 Agent 中承担什么角色
- Reasoning 与 Action 的关系
- 为什么 Agent 的行为具有不确定性

## 3.5 Memory
状态：🟡

需要理解：

- Memory 为什么需要存在
- 短期记忆 / 长期记忆
- Memory 与 Context 的区别
- Memory 的工程实现方式

## 3.6 Reflection
状态：⚪

## 3.7 Multi-Agent
状态：⚪

需要重点理解：

- 为什么需要多个 Agent
- Multi-Agent 与 Workflow 的关系
- Multi-Agent 是否真的比单 Agent 更好

---

# 4. Tools & Agent Extensions

> 目标：理解 Agent 如何获得“行动能力”和外部能力。

## 4.1 Tool
状态：🟡

需要理解：

- Tool 的定义
- Tool 的输入输出
- Tool 与传统 API / Function 的关系

## 4.2 MCP
状态：🟡

需要理解：

- MCP 解决什么问题
- MCP 为什么出现
- MCP Client / Server
- MCP 与 Tool Calling 的关系
- MCP 与 Plugin 的关系
- MCP 的适用边界

## 4.3 Skill
状态：🟡

需要理解：

- Skill 解决什么问题
- Skill 与 Tool 的区别
- Skill 与 Agent 的关系

## 4.4 Plugin
状态：🟡

需要理解：

- Plugin 的基本思想
- Plugin 与 Tool / MCP / Skill 的关系
- 不同 AI Agent 平台中的 Plugin 模型

## 4.5 External Systems
状态：⚪

重点：

- 数据库
- API
- 文件系统
- 浏览器
- 企业业务系统
- 开发工具

---

# 5. Knowledge / RAG

> 目标：理解 Agent / AI Application 如何获得外部知识。

## 5.1 RAG
状态：🟡

需要理解：

- RAG 解决什么问题
- 为什么不能只依赖模型参数
- RAG 基本流程

## 5.2 Retrieval
状态：⚪

## 5.3 Embedding
状态：🟡

## 5.4 Vector Database
状态：⚪

## 5.5 Reranking
状态：⚪

## 5.6 Knowledge Base
状态：⚪

需要理解：

- Knowledge Base 与 RAG 的关系
- 企业知识库的基本架构

---

# 6. Workflow

> 目标：理解确定性流程与 Agent 自主决策之间的边界。

## 6.1 Workflow
状态：🟡

## 6.2 Agent Workflow
状态：⚪

## 6.3 Workflow vs Agent
状态：🔴

重点形成明确判断：

```text
确定性流程 → Workflow
动态决策   → Agent
二者结合   → Agentic Workflow
```

## 6.4 Human-in-the-loop
状态：⚪

## 6.5 Event-driven Agent
状态：⚪

---

# 7. Agent Engineering

> 目标：从“能跑 Agent”进入“能把 Agent 做成可靠的软件系统”。

## 7.1 State Management
状态：⚪

## 7.2 Context Management
状态：🟡

## 7.3 Agent Evaluation
状态：⚪

需要理解：

- 如何判断 Agent 是否做得好
- Evaluation Dataset
- 自动评估
- 人工评估
- 任务成功率

## 7.4 Observability
状态：⚪

## 7.5 Reliability
状态：⚪

重点：

- 重试
- 超时
- Tool Failure
- Model Failure
- 非确定性

## 7.6 Security
状态：⚪

重点：

- Prompt Injection
- Tool 权限
- 数据泄露
- Agent 权限边界

## 7.7 Cost & Performance
状态：⚪

## 7.8 Production Architecture
状态：⚪

---

# 8. AI Coding

> 目标：理解 AI 如何进入软件研发过程，以及 AI Coding 工具背后的 Agent 工程思想。

## 8.1 AI Coding 基本模式
状态：🟡

## 8.2 Claude Code
状态：🟡

重点：

- Agent Loop
- Tools
- Skills
- MCP
- Context
- Coding Workflow

## 8.3 OpenCode
状态：🟡

重点：

- Agent
- Tools
- Skills
- MCP
- 模型适配
- 开源 Agent Coding 架构

## 8.4 DeepSeek Harness
状态：🟡

重点：

- Harness 的基本定位
- Agent / Coding Agent 的运行模式
- Plugin 能力
- 与 Claude Code CLI 的关系
- AI Coding 工具的架构抽象

## 8.5 AI Coding Workflow
状态：🟡

逐步建立：

```text
需求
 ↓
AI 分析
 ↓
代码理解
 ↓
方案设计
 ↓
编码
 ↓
测试
 ↓
Review
 ↓
迭代
```

## 8.6 Agentic Software Engineering
状态：⚪

目标：

- 理解 AI Coding 从“代码补全”向“软件工程 Agent”演进的趋势
- 理解 Agent 在真实研发流程中的边界

---

# 9. AI 工程架构

> 目标：把前面的知识组合起来，形成完整系统设计能力。

## 9.1 AI Application Architecture
状态：⚪

## 9.2 Agent Architecture
状态：⚪

## 9.3 Tool Architecture
状态：⚪

## 9.4 Knowledge Architecture
状态：⚪

## 9.5 Memory Architecture
状态：⚪

## 9.6 Multi-Agent Architecture
状态：⚪

## 9.7 Enterprise AI Architecture
状态：⚪

重点逐步回答：

> 如果让我从零设计一个企业级 AI Agent 系统，我应该如何拆分？

---

# 10. 模型层

> 本层暂时不作为主要学习路线，而是在 AI 工程能力建立后逐步深入。

## 10.1 Transformer
状态：⚪

## 10.2 Attention
状态：⚪

## 10.3 Pre-training
状态：⚪

## 10.4 Fine-tuning
状态：⚪

## 10.5 RLHF / Preference Optimization
状态：⚪

## 10.6 Inference
状态：⚪

## 10.7 Quantization
状态：⚪

## 10.8 Model Serving
状态：⚪

---

# 11. 技术选型能力

> 目标：不追逐“最新技术”，而是能够根据实际问题进行判断。

面对任何新技术，按照：

```text
它是什么？
 ↓
解决什么问题？
 ↓
为什么出现？
 ↓
属于知识地图哪一层？
 ↓
与已有技术是什么关系？
 ↓
有什么优势？
 ↓
有什么代价？
 ↓
什么时候应该使用？
 ↓
什么时候不应该使用？
 ↓
我需要学习到什么程度？
```

学习深度分为：

- **A：深入掌握**
- **B：能够应用**
- **C：理解概念**
- **D：当前阶段暂不投入**

---

# 12. 当前学习路线

V1 不按照技术发布时间学习，而按照认知依赖关系学习：

```text
LLM 基础
   ↓
Prompt / Structured Output
   ↓
Tool Calling
   ↓
Context Engineering
   ↓
RAG / Knowledge
   ↓
Agent
   ↓
Agent Loop / Planning / Memory
   ↓
Workflow
   ↓
MCP / Skill / Plugin
   ↓
Agent Engineering
   ↓
AI Coding
   ↓
AI 工程架构
```

模型原理作为贯穿式补充：

```text
遇到实际问题
   ↓
发现需要模型原理
   ↓
补充 Transformer / Attention / Training / Inference 等知识
```

---

# 13. 阶段性能力目标

## Phase 1：建立 AI 工程整体认知
状态：🟡

目标：

- 能解释 LLM、AI Application、Agent 的关系
- 能理解 Tool Calling
- 能理解 Context
- 能区分 Workflow 与 Agent

## Phase 2：建立 Agent 能力体系
状态：🟡

目标：

- 理解 Agent Loop
- 理解 Tool / MCP / Skill
- 理解 Memory
- 理解 Planning
- 理解 Multi-Agent

## Phase 3：建立 Agent 工程能力
状态：⚪

目标：

- Evaluation
- Observability
- Reliability
- Security
- Cost
- Production

## Phase 4：建立 AI Coding 能力
状态：🟡

目标：

- 理解 Claude Code / OpenCode / DeepSeek Harness
- 理解 Coding Agent 的基本架构
- 建立 AI Coding Workflow
- 探索 Agentic Software Engineering

## Phase 5：形成 AI 工程架构能力
状态：⚪

目标：

> 能够从业务问题出发，独立判断应该使用 LLM、RAG、Tool、Workflow、Agent、Multi-Agent 中的哪些能力，并完成整体架构设计。

---

# 14. 知识地图维护规则

每完成一个重要知识点，都应该更新：

1. **定义**
2. **解决的问题**
3. **所在层级**
4. **核心原理**
5. **与其他知识的关系**
6. **典型工程场景**
7. **适用边界**
8. **当前掌握状态**
9. **下一步学习方向**

不要把所有聊天内容直接复制到本文件。

> **聊天负责探索，本文件负责沉淀。**

当发现原有知识分类不合理时，允许重新调整整个知识地图，而不是为了保持旧结构而强行添加新节点。

---

# 15. 最终目标

最终形成：

```text
AI 工程知识地图
       +
AI 技术判断能力
       +
Agent 工程能力
       +
AI Coding 能力
       +
AI 系统架构能力
```

最终目标不是“学会多少 AI 技术”，而是：

> **面对一个真实的软件工程问题时，能够判断 AI 是否适合解决、应该使用什么 AI 能力、如何组合这些能力，以及如何把它工程化落地。**
