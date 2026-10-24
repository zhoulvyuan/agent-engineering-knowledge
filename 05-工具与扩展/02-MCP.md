# MCP / Model Context Protocol

> **状态**：🟢 已理解 ｜ **深度**：A 深入掌握 ｜ **更新**：2026-09-15
> **上级**：[05-工具与扩展](README.md) ｜ **依赖**：Tool、Tool Calling

## 一句话定义

> MCP（Model Context Protocol，模型上下文协议）是一套开放的客户端—服务器通信标准，它用统一的 JSON-RPC 消息格式规定了 LLM 应用如何发现并调用外部工具、读取外部资源、使用预设提示词，从而把"模型接外部能力"从各家私有集成变成可复用的标准接口。

## 解决的问题

> 在 MCP 出现之前，每个 AI 应用（如某个聊天客户端、IDE 插件、Agent 框架）要接入一个外部能力（数据库、文件系统、GitHub、Slack 等），都得为该应用单独写一份胶水代码。N 个应用 × M 个数据源 = N×M 份互不兼容的集成，重复劳动严重、无法复用、维护成本高。工具的描述格式、鉴权方式、调用约定在每个框架里都不一样，社区生态被割裂成一个个孤岛。MCP 的目标就是把它收敛成 N+M：应用只实现一次 MCP Client，数据源只实现一次 MCP Server，二者即可自由组合。

## 待回答的问题

- [x] MCP 解决什么问题
- [x] MCP 为什么出现
- [x] MCP Client / Server 的角色划分
- [x] MCP 与 Tool Calling 的关系
- [x] MCP 与 Plugin 的关系
- [x] MCP 的适用边界（什么时候不需要 MCP）

### MCP 解决什么问题

- 核心是消除 **M×N 集成爆炸**：把"每个 AI 应用都要为每个数据源单独写集成"收敛为"应用实现一次 Client、数据源实现一次 Server"的 M+N 模式。
- 统一了工具/资源的**描述、发现、调用、鉴权**约定，使同一个 Server（如 GitHub Server）可被任意支持 MCP 的宿主（Claude Desktop、Cursor、各类 Agent 框架）直接复用。
- 把能力集成从"框架内部耦合的代码"变成"进程外、可独立部署和升级的标准服务"，解耦了模型供应商、应用开发者与工具开发者三方。

### MCP 为什么出现

- 2024 年底由 Anthropic 提出并开源，直接动机是 LLM 已能做 Tool Calling，但"模型知道要调工具"和"宿主应用怎样把工具接进来"之间缺一层标准，导致生态碎片化。
- 类比：MCP 想成为"AI 应用接外部世界的 USB-C / LSP"。正如 LSP（Language Server Protocol）让一个语言服务器服务所有编辑器，MCP 让一个工具服务器服务所有 AI 宿主。
- 它顺应了 Agent 工程化趋势：当工具数量、来源、团队都在膨胀时，私有集成不可持续，必须有开放协议承载长尾生态。

### MCP Client / Server 的角色划分

| 角色 | 职责 | 典型实体 |
| --- | --- | --- |
| Host（宿主） | 运行 LLM、管理会话与用户交互、决定调用哪些能力 | Claude Desktop、Cursor、IDE、Agent 应用 |
| Client（客户端） | 由 Host 内部为每个 Server 建立的连接器，与某个 Server 一对一维持会话、转发消息 | Host 内置的 MCP Client 实例 |
| Server（服务端） | 对外暴露 Tools（可执行动作）、Resources（可读数据）、Prompts（预设模板）三类能力 | 文件系统 Server、GitHub Server、数据库 Server |

- 关系是 Host 内含多个 Client，每个 Client 连接一个 Server，一对一会话。
- 通信走 JSON-RPC 2.0，传输层常见两类：本地的 **stdio**（Server 作为子进程）、远程的 **HTTP + SSE / Streamable HTTP**。
- Server 可向 Client 反向请求能力，例如 **Sampling**（让宿主代为调用 LLM）、**Roots**（告知可访问的文件/URI 边界）。

### MCP 与 Tool Calling 的关系

- 二者在不同层：**Tool Calling** 是模型层能力——模型根据工具 schema 输出"要调用哪个函数、传什么参数"的结构化意图；**MCP** 是应用层协议——规定工具怎样被发现、描述、从进程外取回并执行。
- 典型链路：MCP Server 通过 `tools/list` 暴露工具 schema → Host 把 schema 注入给模型 → 模型用 Tool Calling 产出调用意图 → Host 经 MCP Client 发 `tools/call` 给 Server 执行 → 结果回灌模型。
- 没有 MCP 也能做 Tool Calling（直接在代码里硬编码函数）；MCP 的价值是让这些工具标准化、可跨应用复用，而不是替代 Tool Calling。

### MCP 与 Plugin 的关系

- Plugin（如早期 ChatGPT Plugins 基于 OpenAPI）通常是**绑定某个平台**的扩展机制，由平台定义清单格式和运行方式，生态封闭、跨平台不通用。
- MCP 是**厂商中立的开放协议**，不绑定特定模型或宿主；同一个 Server 可被不同厂商的宿主使用，定位更接近"通用插件标准"而非某平台的插件。
- 能力面也更广：Plugin 多聚焦"调 API"，MCP 显式区分 Tools / Resources / Prompts，并支持 Sampling 等双向交互，覆盖"读数据、给上下文、执行动作"的完整需求。

### MCP 的适用边界（什么时候不需要 MCP）

- 工具数量少且固定、只服务单一应用时，直接用框架原生的函数调用更简单，引入 MCP 反而增加进程、协议、部署的额外复杂度。
- 对延迟极敏感或纯内部的能力，进程外 JSON-RPC 往返的开销不划算。
- 当能力本质是"纯检索知识"时，RAG / 向量检索往往比包成 MCP 工具更直接。
- 一句话：MCP 是为"多宿主复用、长尾生态、工具需独立演进"而生；单体、封闭、少量工具的场景不需要它。

## 核心内容

- **三类能力原语**：Tools（模型可调用的动作，有副作用）、Resources（可读的上下文数据，如文件/记录，用 URI 标识）、Prompts（可复用的提示词模板，常由用户显式触发）。
- **消息协议**：基于 JSON-RPC 2.0，含请求/响应/通知；标准方法如 `initialize`、`tools/list`、`tools/call`、`resources/list`、`resources/read`、`prompts/list`、`prompts/get`。
- **传输层**：本地 **stdio**（Server 作为宿主子进程，适合本机工具）；远程 **Streamable HTTP / HTTP+SSE**（适合托管服务、团队共享）。
- **会话生命周期**：初始化握手协商协议版本与能力（capabilities）→ 能力发现（list）→ 调用/读取（call/read）→ 关闭。
- **双向交互**：Server 可发起 Sampling（借宿主的 LLM 推理）、声明 Roots（文件/URI 访问边界），实现 Client↔Server 双向协作。
- **安全模型**：强调用户知情同意——工具执行、资源访问应经用户确认；鉴权（如远程 Server 的 OAuth）、权限边界与审计是宿主的责任。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | Tool Calling / Function Calling | MCP 承载的 Tools 最终仍由模型的工具调用能力驱动，MCP 是其标准化的集成层 |
| 下游 | Agent 框架与 Agent 应用 | Agent 把 MCP Server 作为可插拔的工具/资源来源，扩展感知与行动能力 |
| 易混淆 | OpenAPI / 插件清单 / LangChain Tools | 都在"接外部能力"，但 MCP 是厂商中立的运行时协议，强调跨宿主复用与 Resources/Prompts/双向交互，而非平台私有或框架内置 |

## 典型工程场景

- IDE / 编码助手（Cursor、Claude Code 等）接入文件系统、Git、终端、数据库 Server，让 Agent 读代码、跑命令、查数据。
- 企业内部把 Jira、Confluence、数据仓库、监控系统各封装成一个 MCP Server，供多个 AI 应用统一复用，而不必各自集成。
- 构建可插拔的 Agent 工具市场：第三方以 MCP Server 形式发布能力，用户在任意支持 MCP 的宿主里一键挂载。
- 客服 / 运维 Agent 通过 MCP 连接工单系统、知识库（Resources）与操作接口（Tools），实现查询+执行闭环。

## 适用边界

**适合使用**

- 同一批工具/数据源需要被**多个 AI 宿主或团队复用**。
- 工具生态**长尾、频繁增减**，希望工具能独立部署、独立升级、与主应用解耦。
- 需要进程/权限隔离，让外部能力在受控边界内运行并可审计。
- 希望借助已有的开源 Server 生态快速接入常见系统（文件、Git、DB、SaaS）。

**不适合使用**

- 单一应用、少量且固定的内部函数——原生函数调用更轻量。
- 对延迟极度敏感、不能承受进程外 RPC 往返开销的路径。
- 纯知识检索场景，RAG 通常比包成工具更直接。
- 安全/合规要求禁止引入外部进程或第三方 Server，而又无力自建受控 Server 时。

## 常见误区

- 把 MCP 当成"模型新能力"：它不提升模型推理，只是标准化工具/上下文的接入方式。
- 以为 MCP 替代 Tool Calling：实为互补，MCP 依赖模型的工具调用能力来触发 Tools。
- 以为 MCP 只有工具：忽略了 Resources（读数据）和 Prompts（模板）两类原语，以及 Sampling 等双向交互。
- 忽视安全：直接挂载来源不明的第三方 Server 等于把执行权交给外部，存在数据泄露与命令执行风险，必须做权限、确认与审计。
- 认为所有能力都该做成 MCP：小而封闭的场景强行协议化只会增加复杂度。

## 我的结论

> 该用 MCP 的信号是"复用"和"解耦"：当同一批工具/数据源要服务多个宿主或团队、工具生态长尾且需独立演进、或需要进程与权限隔离时，MCP 用一次标准化换来跨应用的可组合性，收益明显。反之，单体应用里少量固定的内部函数、延迟敏感路径、纯检索需求，直接用原生函数调用或 RAG 更简单，强上 MCP 只是徒增进程与协议开销。一句话：MCP 是"工具生态的标准接口"，面向规模化与可复用而生，不是每个工具都要戴上的帽子；引入前先确认是否真有跨宿主复用或独立演进的需求，并把第三方 Server 的安全边界想清楚。

## 待解决问题

- [ ] Streamable HTTP 传输（新规范）与早期 HTTP+SSE 在生产远程部署、鉴权（OAuth）与可扩展性上的具体差异与迁移影响。
- [ ] 大规模挂载 Server 时，工具数量膨胀对模型选择准确率与上下文占用的影响，以及工具路由/分组的最佳实践。
- [ ] 第三方 MCP Server 的供应链与权限安全治理（签名、沙箱、最小权限、审计）在企业落地的成熟方案。

## 参考资料

- Model Context Protocol 官方文档与规范（modelcontextprotocol.io，含 Specification 与 SDK）
- Anthropic《Introducing the Model Context Protocol》公告（2024）
- JSON-RPC 2.0 Specification
- Language Server Protocol（LSP）规范——MCP 设计理念的类比参考
- 官方开源仓库 github.com/modelcontextprotocol（servers 示例集、SDK）
