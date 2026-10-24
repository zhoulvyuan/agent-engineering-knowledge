# Observability / 可观测性

> **状态**：🟢 已理解 ｜ **深度**：A 深入掌握 ｜ **更新**：2026-09-15
> **上级**：[08-Agent工程化](README.md) ｜ **依赖**：Evaluation、State Management

## 一句话定义

> Agent 可观测性是指把一次 Agent 运行中「模型推理、工具调用、状态变更、Token 与成本」等内部过程，以结构化的 Trace/Span/事件形式记录并可视化出来，使外部能够从数据反推系统内部到底发生了什么、为什么会这样。

## 解决的问题

> 在它普及之前，Agent 对工程师基本是「黑盒」：LLM 的决策是非确定的，一次任务常由多轮循环、多次工具调用、多段上下文拼接组成，出问题时只能看到「最终答案不对」，却无法还原中间哪一步跑偏。日志通常是零散的 print/文本，缺乏请求级别的因果串联，无法回答「这次失败花了多少 Token、调了几次工具、哪一步的 Prompt 导致模型幻觉」。同时成本不可见——线上跑了一周才发现账单爆炸却定位不到是哪类请求。可观测性把这些隐性过程变成可查询、可聚合、可回放的数据，让调试、归因、成本治理和线上监控成为可能。

## 待回答的问题

- [x] 一次 Agent 运行需要记录什么
- [x] Trace / Span 如何组织（一次任务 / 一个循环 / 一次工具调用）
- [x] OpenTelemetry GenAI 语义约定
- [x] Token 与成本的实时追踪
- [x] 可观测工具选型（Langfuse / LangSmith 等）
- [x] 线上问题如何定位到具体循环步骤

### 一次 Agent 运行需要记录什么

至少应覆盖「输入—决策—动作—结果—度量」五类数据：

- 输入与上下文：原始用户请求、系统提示、注入的记忆/检索片段、最终拼好的完整 Prompt（含消息角色）。
- 模型调用：模型名与版本、采样参数（temperature/top_p）、请求与响应原文、finish_reason、函数/工具调用的结构化参数。
- 工具/外部调用：工具名、入参、返回值、耗时、是否报错、重试次数。
- 控制流与状态：当前循环轮次、Agent 的中间 reasoning/plan、状态机/记忆的读写快照。
- 度量与元数据：各步耗时与端到端延迟、prompt/completion/total Token、单价换算出的成本、user_id/session_id/trace_id、环境与代码版本。

关键原则是记录到「可因果回放」的粒度——理想情况下能据此离线重放整条链路。注意对 PII、密钥等敏感字段做脱敏或采样。

### Trace / Span 如何组织（一次任务 / 一个循环 / 一次工具调用）

采用树状/嵌套结构，和分布式追踪一致：

| 层级 | 对应概念 | 粒度 |
| --- | --- | --- |
| Trace（根） | 一次完整任务/会话请求 | 一个用户目标的端到端执行 |
| Span（中层） | 一轮 Agent 循环、一次规划、一次子 Agent 调用 | ReAct 的一个 thought-action-observation 周期 |
| Span（叶子） | 一次 LLM 调用、一次工具调用、一次检索 | 单个原子操作 |

每个 Span 带有 start/end 时间、父子关系、attributes（输入输出、Token、状态）和 status（成功/异常）。通过统一的 trace_id 串联，span 之间以 parent_span_id 形成因果树，这样既能看端到端耗时瀑布图，也能逐层下钻到某次具体工具调用。嵌套而非扁平日志，是定位「第几轮循环出问题」的前提。

### OpenTelemetry GenAI 语义约定

OpenTelemetry 在传统 trace/metric/log 之上，为 LLM/GenAI 场景定义了一套语义约定（Semantic Conventions for Generative AI），统一属性命名，便于跨厂商工具互通：

- 典型 span 属性：`gen_ai.system`（如 openai）、`gen_ai.request.model`、`gen_ai.request.temperature`、`gen_ai.response.model`、`gen_ai.response.finish_reasons`。
- Token 用量：`gen_ai.usage.input_tokens`、`gen_ai.usage.output_tokens`。
- 操作类型：`gen_ai.operation.name`（chat、embeddings、tool 等）。
- 提示与补全内容过去放在 span events，现逐步迁移到独立的 log/event 以控制体积与隐私。

注意该规范仍处于演进/部分 experimental 状态，属性名称与承载方式会调整；选型时应确认所用 SDK 对应的约定版本。采用 OTel 的价值在于避免被单一 APM/LLM 平台锁定——数据可同时导出到 Jaeger、Grafana、Langfuse 等后端。

### Token 与成本的实时追踪

实时成本追踪的核心是「在每次模型调用的 span 上记录 token 用量，再按模型单价换算并逐层聚合」：

- 来源优先用 API 返回的 usage 字段（最准确）；对不返回 usage 的流式场景可用 tokenizer 本地估算。
- 维护一张「模型→输入/输出单价」映射表，成本 = input_tokens×单价in + output_tokens×单价out；注意缓存命中、batch、不同区域价格差异。
- 聚合维度：按 trace（单次任务成本）、session/user（人均成本）、feature/tenant（业务线成本）累加。
- 实时性做法：span 上报后由后端流式聚合，配合看板与告警阈值（如单请求超 N 美元、日成本环比突增）即时发现异常。

它同时服务于两个目的：定位「哪类请求烧钱」和为限流/配额/计费提供数据基础。

### 可观测工具选型（Langfuse / LangSmith 等）

| 工具 | 定位 | 特点与适用 |
| --- | --- | --- |
| Langfuse | 开源 LLM 可观测平台 | 可自托管、支持 OTel，trace+评估+提示管理+成本，重数据主权时首选 |
| LangSmith | LangChain 官方平台 | 与 LangChain/LangGraph 深度集成、调试与数据集评估体验好，偏 SaaS |
| Arize Phoenix | 开源，侧重评测与排障 | OTel 原生、本地可跑，适合实验与 RAG 质量分析 |
| Helicone / W&B Weave 等 | 代理网关 / 实验追踪 | Helicone 以反向代理方式低侵入接入 |
| 通用 APM（Grafana/Datadog/Jaeger） | 基础设施级 | 已有 OTel 栈时把 GenAI span 并入统一监控 |

选型主线：是否需要自托管/数据合规（Langfuse、Phoenix 占优）；是否已绑定某框架（LangChain→LangSmith）；是否要与现有 OTel/APM 融合（选 OTel 原生者）；以及是否需要内建评估与数据集回归。注意各平台功能与定价迭代快，具体能力以当期官方文档为准。

### 线上问题如何定位到具体循环步骤

标准排障路径：

1. 以 trace_id 为锚点——在返回/日志/告警中带上 trace_id，用户报障时据此直达该次执行。
2. 看瀑布图定位异常 span：哪一轮循环耗时异常、哪次工具调用报错、哪次 LLM 调用 finish_reason=length 被截断。
3. 下钻到该 span 的输入输出：检查当轮完整 Prompt、模型原始输出、工具入参/返回，判断是幻觉、上下文污染、工具失败还是解析错误。
4. 结合状态快照看跨轮因果：常见根因是前一轮把错误信息写入记忆/状态，导致后续循环连环出错。
5. 必要时离线回放该 trace，或将其加入评估数据集做回归，防止复发。

前提是记录粒度足够细且 span 嵌套正确——否则只能看到「任务失败」而无法下钻。

## 核心内容

- 三支柱映射到 Agent：Trace（请求级因果链）、Metrics（延迟/Token/成本/成功率等聚合指标）、Logs（结构化事件），GenAI 场景以 Trace 为核心。
- 嵌套 Span 模型：Trace→循环 Span→LLM/工具叶子 Span，靠 trace_id + parent_span_id 构成因果树。
- 结构化而非文本日志：记录输入、Prompt、模型输出、工具调用、Token、成本、状态快照等可回放数据。
- 标准化：OpenTelemetry GenAI 语义约定统一属性命名，支持多后端导出、避免厂商锁定。
- 成本与性能度量：实时聚合 token/成本/延迟，支撑看板、告警与计费。
- 评估闭环：线上 trace 可沉淀为数据集，驱动离线评估与回归（与 Evaluation 强耦合）。
- 隐私与采样：对敏感数据脱敏、对高流量按比例采样以控制存储与合规风险。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | State Management、Agent Loop（ReAct/规划循环） | 可观测性记录的正是循环每一步的状态读写与决策；没有清晰的循环与状态结构，就无法切出有意义的 Span |
| 下游 | Evaluation、成本治理与告警、线上运维/排障 | 线上 trace 沉淀为评估数据集驱动回归；成本/延迟指标支撑告警、限流与计费决策 |
| 易混淆 | 传统 APM 监控、普通日志（logging）、Evaluation | APM/日志关注系统健康与文本记录，可观测性强调非确定链路的因果回放；Evaluation 回答「好不好」，可观测性回答「发生了什么、为什么」，二者互补而非等同 |

## 典型工程场景

- 调试多轮 Agent：复现「任务中途跑偏」，定位是哪一轮循环、哪个工具或哪段 Prompt 导致。
- 成本治理：按用户/业务线统计 Token 与成本，发现并优化烧钱的高频长上下文请求。
- 线上监控告警：对延迟、错误率、单请求成本、幻觉/截断率设阈值告警。
- RAG 质量排查：检查检索片段是否相关、是否被正确注入上下文、模型是否据此作答。
- 回归与评估：把线上失败 trace 收集成数据集，防止提示或模型升级造成退化。
- 多 Agent/工具编排：观察子 Agent 调用树与工具链路，发现死循环、重复调用、超时。
- 合规审计：留存可追溯的决策记录，满足审计与问责需求。

## 适用边界

**适合使用**

- 任何进入线上、面向真实用户的 Agent/LLM 应用，尤其多轮循环与多工具编排。
- 需要成本管控、SLA 监控、质量回归或审计留痕的生产系统。
- 调试非确定性行为、排查幻觉与上下文问题的开发/联调阶段。

**不适合使用**

- 一次性脚本、原型 demo，用简单日志即可，无需引入完整追踪栈。
- 极端低延迟/资源受限场景下的全量同步采集（应改为异步上报+采样）。
- 把它当成质量评估的替代：它记录「发生了什么」，但不自动判断「好不好」，需配合 Evaluation。
- 直接记录未脱敏的敏感数据（PII、密钥）会带来合规风险，需先做脱敏策略。

## 常见误区

- 把文本 print 日志等同于可观测性：缺乏 trace_id 串联和嵌套结构，无法做因果回放与下钻。
- 只记最终输入输出，不记中间循环/工具/状态，出问题时仍是黑盒。
- 认为上了平台就等于有了质量保障：可观测性≠评估，不会自动告诉你答案对不对。
- Token/成本用本地估算代替 API 返回的 usage，导致账单对不上。
- 全量同步采集高流量链路，拖垮主流程延迟且存储爆炸——应异步、采样、限大小。
- 忽视隐私：把完整 Prompt/响应（含用户隐私）原样入库而不脱敏。
- 被单一平台锁定：不走 OTel 标准，后续迁移/多后端困难。

## 我的结论

> 只要 Agent 要上线、要被真实用户使用，就应该从第一天引入可观测性——因为 LLM 的非确定性决定了「不可观测=不可维护」，缺它时调试、归因和成本治理几乎无从下手。落地上优先采用 OpenTelemetry GenAI 语义约定做标准化埋点，再按「是否需自托管/合规、是否已绑定框架、是否要融入现有 APM」选择 Langfuse（重数据主权）、LangSmith（LangChain 生态）或 Phoenix（评测排障）等后端。
> 什么时候不该用：一次性脚本或纯原型阶段，用轻量日志即可，不必过度工程化；也不要把它误当质量评估的替代，它只回答「发生了什么」，判断「好不好」仍要交给 Evaluation。记录粒度要足以因果回放，但对敏感数据脱敏、对高流量采样，避免拖累主流程和踩合规红线。

## 待解决问题

- [ ] OpenTelemetry GenAI 语义约定目前的稳定程度如何，提示/补全内容最终是以 span event 还是独立 log 承载，具体属性名在最新版本是否已变动？
- [ ] 对高 QPS 的 Agent 服务，如何设计采样策略（头采样 vs 尾采样）在成本、存储与「不漏掉失败链路」之间取得平衡？
- [ ] Langfuse / LangSmith / Phoenix 等平台在自托管成本、评估能力、OTel 兼容性上的最新差异如何（功能与定价迭代快，需按当期官方文档复核）？

## 参考资料

- OpenTelemetry — Semantic Conventions for Generative AI Systems（官方规范）
- OpenTelemetry 官方文档：Traces / Spans / Metrics / Logs 基础概念
- Langfuse 官方文档（Tracing、Cost tracking、OTel 集成）
- LangSmith 官方文档（LangChain/LangGraph tracing 与 evaluation）
- Arize Phoenix 官方文档（LLM tracing 与 evaluation）
- 《Observability Engineering》(Charity Majors 等, O'Reilly) — 可观测性通用理论
- Google Dapper 论文（分布式追踪的奠基性工作）
