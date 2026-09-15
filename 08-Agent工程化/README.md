# 08 Agent 工程化

> 主线域 ｜ 目标：从「能跑 Agent」进入「能把 Agent 做成可靠的软件系统」。

## 本域定位

本域是**整个知识库的分水岭**：前面所有域回答"怎么让它工作"，
本域回答"**怎么让它一直可靠地工作**"。

原始地图中本域几乎全是 ⚪，这是**最大的能力空白**，也是"能做 demo"和
"能上生产"之间的全部差距所在。

**给你的 Java 背景的建议**：本域是你**相对优势最大**的地方。
可观测性、可靠性、状态管理、幂等、熔断、灰度发布——这些在传统后端工程里
已经有成熟方法论，你要做的是把 LLM 的非确定性作为**新增变量**叠加进去，
而不是从零学一套东西。

**优先级建议**（不要把本域当成最后才碰的章节）：

- 做完第一个能跑的 Agent 后，**立刻**引入轻量版的可观测与评估
- 否则你会陷入"改了 Prompt，不知道是变好还是变坏"的困境

## 知识点清单

| 知识点 | 状态 | 深度 | 一句话 |
| --- | --- | --- | --- |
| [State Management / 状态管理](01-State-Management.md) | ⚪ | A | Agent 运行状态的保存与恢复 |
| [Context Management / 上下文管理](02-Context-Management.md) | 🟡 | A | 运行时的上下文预算与裁剪 |
| [Agent Evaluation / 评估](03-Evaluation.md) | ⚪ | A | 判断 Agent 好坏的度量体系 |
| [Observability / 可观测性](04-Observability.md) | ⚪ | A | 让 Agent 的运行过程可见 |
| [Reliability / 可靠性](05-Reliability.md) | ⚪ | A | 让 Agent 在异常下仍可预期 |
| [Security / 安全](06-Security.md) | ⚪ | A | Agent 拥有行动能力后的风险控制 |
| [Cost & Performance / 成本与性能](07-Cost与Performance.md) | ⚪ | B | 让 Agent 在经济上可持续 |
| [Production Architecture / 生产架构](08-Production-Architecture.md) | ⚪ | A | 把 Agent 变成可运维的系统 |

## 待建文件（已识别的知识缺口）

这些知识点**已登记但尚未建文件**，学到时再创建，避免产生大量空壳。

| 知识点 | 优先级 | 为什么需要 |
| --- | --- | --- |
| 非确定性下的测试策略 | **高** | 传统单元测试在 LLM 应用中失效，需要断言宽松化、快照测试、评估集驱动的新范式 |
| Prompt / 模型版本管理 | 中 | 变更如何灰度、回滚、对比 |
| Agent 沙箱与代码执行隔离 | 中 | 执行模型生成的代码时的隔离方案 |

## 与其他域的关系

| 域 | 关系 |
| --- | --- |
| 03-Agent核心 | 本域是其生产化 |
| 07-Workflow编排 | 状态与可靠性在本域统一 |
| 09-AI-Coding | Coding Agent 是本域思想的集中体现 |
| 11-架构设计 | 本域是架构设计的约束条件 |

## 建议学习顺序

1. **State Management** — 地基，长任务的前提
2. **Context Management** — 与 Context Engineering 的区别在于这是**运行时**的预算管理
3. **Evaluation** — 没有评估就没有迭代方向
4. **Observability** — 没有观测就无法定位问题
5. **Reliability** — 异常处理与降级
6. **Security** — 有行动能力就必须谈安全
7. **Cost & Performance** — 经济性
8. **Production Architecture** — 综合落地

> 建议 3、4 两个节点在学习第一个 Agent 后就提前接触。

## 更新记录

| 日期 | 变更 |
| --- | --- |
| 2026-09-15 | 建立本域骨架 |
