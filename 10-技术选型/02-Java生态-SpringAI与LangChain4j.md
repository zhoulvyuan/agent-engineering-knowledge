# Java 生态：Spring AI 与 LangChain4j

> **状态**：🟢 已理解 ｜ **深度**：B 能够应用 ｜ **更新**：2026-09-15
> **上级**：[10-技术选型](README.md) ｜ **依赖**：技术选型方法论

## 一句话定义

Spring AI 与 LangChain4j 是运行在 JVM 上的两套 AI 应用开发框架，它们把大模型调用、提示工程、嵌入与向量检索、工具调用、RAG、Agent 编排等能力封装成 Java API，让 Java/Kotlin 开发者无需跳到 Python 就能在既有企业技术栈内构建生成式 AI 应用。

## 解决的问题

在它们出现之前，Java 团队要接入大模型通常面临几类麻烦：

- **语言割裂**：LLM 生态几乎都以 Python 为中心（LangChain、LlamaIndex），Java 团队要么另起一套 Python 服务、承担跨语言 RPC 与运维成本，要么手写 HTTP 客户端裸调各家 API。
- **厂商锁定与适配成本**：OpenAI、Azure、Anthropic、Ollama、各类国产模型的接口、鉴权、流式协议、参数命名各不相同，每换一家都要重写一遍胶水代码。
- **样板代码重复**：提示模板拼接、对话历史管理、Token 计数、重试与超时、向量库读写、函数/工具调用的 JSON Schema 生成等，缺乏统一抽象，人人重复造轮子。
- **难以融入企业既有体系**：Java 团队已有成熟的依赖注入、配置管理、可观测性（Micrometer）、事务、安全框架，裸调 API 无法自然复用这些能力。

## 待回答的问题

- [x] Spring AI 和 LangChain4j 各自定位与差异
- [x] 对 Java 背景开发者的价值（复用哪些既有能力）
- [x] 它们覆盖/不覆盖哪些 AI 能力
- [x] 什么场景选 Java 栈、什么场景仍该用 Python

### Spring AI 和 LangChain4j 各自定位与差异

两者目标相近，但出身与风格不同：

- **Spring AI**：由 Spring 官方团队出品，定位是"Spring 生态里的 AI 模块"，设计哲学与 Spring Data、Spring Boot 一脉相承，强调自动配置（starter）、`ChatClient` 流式 API、Bean 注入、`application.yml` 配置化、Observability 等。它天然服务于已经深度使用 Spring Boot 的团队。
- **LangChain4j**：社区驱动的独立框架，灵感来自 Python 的 LangChain，定位是"框架无关的 Java AI 工具箱"。它不强制绑定 Spring，可在纯 Java、Quarkus、Micronaut 等任意环境使用，同时也提供 Spring Boot 与 Quarkus 的 starter。它的 `AiServices`（声明式接口代理）和链式组合能力是特色。

对比：

| 维度 | Spring AI | LangChain4j |
| --- | --- | --- |
| 出品方 | Spring 官方 | 社区（独立项目） |
| 耦合取向 | 深度绑定 Spring Boot | 框架无关，可选 Spring/Quarkus |
| 核心抽象 | `ChatClient`、`EmbeddingModel`、`VectorStore`、`Advisor` | `ChatModel`、`AiServices`、`EmbeddingStore`、`Tool` |
| 声明式风格 | Fluent API 为主 | 接口 + 注解生成代理（类似 Feign） |
| 配置方式 | Spring Boot 自动配置 + yml | 代码构建为主，亦有 starter |
| 适合团队 | 已在用 Spring 全家桶 | 想要轻量/非 Spring 或需要更细粒度控制 |

一句话：**Spring AI 是"Spring 原生"，LangChain4j 是"Java 通用"**；功能覆盖高度重叠，选择更多取决于团队现有栈与风格偏好。

### 对 Java 背景开发者的价值（复用哪些既有能力）

对 Java 团队，核心价值是"不离开熟悉的工程体系就能做 AI"：

- **依赖注入与配置**：模型客户端、向量库、嵌入模型都是 Bean，可注入、可按环境配置、可通过 Profile 切换（Spring AI 尤为自然）。
- **可观测性**：复用 Micrometer / Tracing，把 Token 用量、调用延迟、链路追踪接入既有 Prometheus/Grafana/OTel 体系，而不是另搭一套 Python 监控。
- **类型安全与结构化输出**：利用 Java 的强类型，把 LLM 输出直接映射为 POJO（Structured Output / `AiServices` 返回自定义类型），编译期就能约束结构。
- **既有中间件生态**：事务、连接池、消息队列（Kafka）、缓存、安全（Spring Security）、测试（JUnit/Testcontainers）全部可直接复用。
- **部署与运维一致性**：产物仍是一个 JAR/容器，走原有 CI/CD、灰度、扩缩容流程，无需为 AI 单独引入 Python 运行时与打包方式。

### 它们覆盖/不覆盖哪些 AI 能力

**覆盖（应用编排层）**：

- 多厂商 Chat / Completion 模型调用，支持同步与流式（SSE/Reactive）。
- 提示模板、对话记忆（Chat Memory）、消息角色管理。
- 嵌入（Embedding）生成与向量库对接（PGVector、Redis、Milvus、Qdrant、Elasticsearch、Chroma 等）。
- RAG：文档加载、切分、嵌入、检索、重排、上下文注入的完整链路。
- 工具/函数调用（Function/Tool Calling）、结构化输出映射到 POJO。
- 多模态（图像/音频，视模型而定）、基础的 Agent 与多步推理编排。

**不覆盖（或非其所长）**：

- **模型训练、微调、推理服务本身**：它们是"调用方/编排方"，不负责训练 LLM、不实现推理引擎（那是 PyTorch、vLLM、TGI 的职责）。
- **前沿研究算法的第一时间实现**：新论文方法、实验性技巧通常先在 Python 社区落地。
- **深度数据科学/特征工程**：缺乏 pandas、NumPy 级别的数据处理生态。
- **复杂多 Agent 图编排的成熟度**：相较 Python 的 LangGraph 等，JVM 侧的高级 Agent 编排仍在演进。

### 什么场景选 Java 栈、什么场景仍该用 Python

| 维度 | 倾向 Java 栈（Spring AI / LangChain4j） | 倾向 Python 栈 |
| --- | --- | --- |
| 团队现状 | 核心系统已是 Java/Spring，团队熟 JVM | 团队以数据科学/ML 为主，熟 Python |
| 应用形态 | 企业后端、需融入现有事务/安全/监控的业务系统 | 研究原型、Notebook 实验、独立 AI 服务 |
| 能力需求 | 模型调用、RAG、工具调用等"应用编排" | 训练、微调、自定义推理、前沿算法 |
| 生态依赖 | 需复用 Java 中间件与运维体系 | 需 HuggingFace、PyTorch、pandas 深度生态 |
| 迭代速度 | 稳定长期维护、强类型可靠性优先 | 快速试错、紧跟最新模型特性优先 |

经验法则：**"应用层编排在哪端成本更低就在哪端做"**。业务后端是 Java 就用 Java 栈接 AI；凡涉及训练/微调/重数据处理/前沿算法，就交给 Python 侧服务，再通过 API 协作。

## 核心内容

两框架的关键机制可归纳为几层抽象：

**1. 模型抽象层**

- 统一接口屏蔽厂商差异：Spring AI 的 `ChatModel`/`ChatClient`、`EmbeddingModel`；LangChain4j 的 `ChatModel`/`StreamingChatModel`、`EmbeddingModel`。
- 通过 starter 或 builder 配置具体 provider（OpenAI、Azure、Anthropic、Ollama、Bedrock、国产模型等），换模型基本只改配置不改业务代码。

**2. 提示与记忆层**

- 提示模板（PromptTemplate）支持变量占位与多消息组装（system/user/assistant）。
- Chat Memory 管理多轮对话历史，可按会话隔离、按 Token 窗口裁剪。

**3. 结构化输出 / 工具调用**

- 结构化输出：把 LLM 文本解析为强类型 POJO（基于 JSON Schema 约束）。
- 工具调用：把 Java 方法声明为 `@Tool`（LangChain4j）或 Function（Spring AI），框架自动生成 Schema、处理模型回传的调用请求并执行方法，实现"模型驱动代码执行"。

**4. RAG 流水线**

典型流程：

```
文档加载 → 切分(Chunking) → 嵌入(Embedding) → 存入向量库(VectorStore)
                                                        │
用户提问 → 嵌入 → 相似检索(Top-K) → (可选重排) → 组装上下文 → 调用 LLM → 回答
```

- Spring AI 用 `Advisor`（如 `QuestionAnswerAdvisor`）把检索逻辑挂到 `ChatClient` 调用链上。
- LangChain4j 用 `RetrievalAugmentor` / `ContentRetriever` 组合检索与增强。

**5. 声明式服务（LangChain4j 特色）**

- `AiServices` 让你定义一个 Java 接口 + 注解，框架在运行时生成代理实现，自动完成提示拼装、记忆、工具、输出解析——风格类似用 Feign 声明 HTTP 客户端。

**6. 可观测性与弹性**

- 集成 Micrometer 指标与分布式追踪；配合重试、超时、限流保障生产可用性。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | 大语言模型 API（OpenAI/Anthropic/Ollama 等）、向量数据库、Embedding 模型 | 两框架是这些能力的"调用方与编排方"，底层能力由模型服务与向量库提供 |
| 下游 | 企业 AI 业务应用（智能客服、知识问答、文档助手、Copilot 类后端） | 实际交付给用户的应用建立在框架提供的编排能力之上 |
| 易混淆 | Python 版 LangChain / LlamaIndex、Spring AI 与 LangChain4j 互相之间 | 功能高度重叠易混；关键区分是运行时（JVM vs Python）与耦合取向（Spring 原生 vs 框架无关） |

## 典型工程场景

- **企业知识库问答（RAG）**：把内部文档、工单、Wiki 嵌入向量库，为员工或客户提供带引用的问答，直接复用已有的 Spring Security 做权限隔离。
- **智能客服 / 工单辅助**：在既有 Java 客服系统中接入对话模型，结合工具调用查订单、改状态。
- **业务系统内嵌 Copilot**：在 ERP/CRM/运维平台中加入自然语言操作入口，通过 Tool Calling 调用后端服务。
- **文档/报表自动化**：合同摘要、信息抽取（结构化输出到 POJO 直接入库）。
- **多模型路由与成本控制**：按场景在不同模型间切换（便宜模型兜底、强模型处理复杂请求），复用 Micrometer 监控 Token 成本。

## 适用边界

**适合使用：**

- 主技术栈是 Java/Kotlin、Spring Boot，希望 AI 能力融入既有工程与运维体系。
- 需求集中在"应用编排"：模型调用、RAG、工具调用、对话管理。
- 看重强类型、长期可维护性、与企业中间件的一致性。
- 需要把 AI 功能作为现有后端服务的一部分平滑上线。

**不适合使用：**

- 需要模型训练、微调或自建推理引擎——应交给 Python/专用推理服务。
- 以数据科学、特征工程、实验探索为主——Python 生态更顺手。
- 必须第一时间用上最前沿算法/模型新特性——Java 侧适配通常滞后。
- 纯粹的一次性脚本或 Notebook 原型，引入 JVM 框架反而是负担。

## 常见误区

- **"用了框架就不懂底层也能做好 AI"**：框架只降低胶水成本，提示工程、RAG 质量、评测仍需认真对待。
- **"Spring AI 和 LangChain4j 必须二选一且差别巨大"**：两者能力高度重叠，差异主要在风格与耦合；选择更多看团队现状，而非功能缺失。
- **"Java 栈能替代 Python 做一切 AI 工作"**：它们不覆盖训练/微调/重数据处理，定位是应用编排层。
- **"换模型零成本"**：接口虽统一，但不同模型的能力、上下文窗口、工具调用支持、提示敏感度不同，切换后仍需重新评测与调优。
- **"RAG 加上向量库就准了"**：检索质量取决于切分策略、嵌入模型、Top-K、重排与提示组织，框架只是提供了管道。
- **"框架版本稳定可随意升级"**：两者仍在快速演进，API 可能变动，生产使用需锁定版本并关注变更。

## 我的结论

**何时该用**：当团队的核心系统已经是 Java/Spring，且 AI 需求集中在"把大模型能力编排进业务后端"（RAG、对话、工具调用、结构化抽取）时，Java 栈是更低总成本的选择——它让 AI 功能复用既有的注入、配置、安全、事务与可观测性，避免为此单独维护一套 Python 服务。其中已重度使用 Spring Boot 的团队优先 Spring AI；希望框架无关或偏好声明式 `AiServices` 风格的，选 LangChain4j。

**何时不该用**：当工作重心是模型训练、微调、自定义推理、重度数据处理，或需要紧跟最前沿研究时，仍应使用 Python 栈，并通过 API 让 Java 应用与之协作。两者不是竞争而是分工：**Python 造与调模型，Java 把模型能力稳稳地嵌进企业系统**。

## 待解决问题

- [ ] 在高级多 Agent / 图式编排（对标 Python LangGraph）上，Spring AI 与 LangChain4j 的成熟度与最佳实践目前到什么程度？
- [ ] 两框架在大规模并发、流式长会话下的性能与资源表现，以及与响应式栈（WebFlux/Reactor）的最佳集成方式如何？
- [ ] 面向生产的 LLM 评测、回归与质量保障，在 Java 生态里有哪些可落地的工具与模式？

## 参考资料

- Spring AI 官方文档（Spring 项目站点 docs.spring.io/spring-ai）
- LangChain4j 官方文档与 GitHub 项目（langchain4j）
- Spring Boot / Spring Framework 官方参考文档
- Micrometer 可观测性文档
- 各模型厂商 API 文档（OpenAI、Anthropic、Azure OpenAI、Ollama）
- 常见向量数据库文档（PGVector、Milvus、Qdrant、Redis、Elasticsearch）
- Python LangChain / LlamaIndex 文档（作对比参考）
