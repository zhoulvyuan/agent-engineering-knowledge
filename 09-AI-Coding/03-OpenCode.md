# OpenCode

> **状态**：🟢 已理解 ｜ **深度**：B 能够应用 ｜ **更新**：2026-09-15
> **上级**：[09-AI-Coding](README.md) ｜ **依赖**：Agent Loop

## 一句话定义

> OpenCode 是一个开源、provider-agnostic（不绑定模型厂商）的终端 AI 编码 Agent，采用「客户端/服务端」架构：核心运行时跑一个本地 server，TUI/Web/IDE 等多种前端作为客户端接入，让 LLM 通过一组内置工具在你的代码库里自主读写、执行命令并完成编码任务。

## 解决的问题

> 在 OpenCode 这类工具出现前，开发者用 LLM 写代码主要靠「复制粘贴」：在聊天窗口贴代码、拿回答、再手工改回文件，LLM 看不到真实仓库、跑不了测试、改不了多文件。后来出现的闭源方案（如 Claude Code、Cursor、Copilot）又把能力锁定在特定厂商模型和特定编辑器里，难以自由切换模型、难以二次开发、也不方便在服务器/远程环境中以纯终端方式运行。OpenCode 要解决的就是：让一个「能真正操作仓库的编码 Agent」变成开源、可自托管、可换任意模型、可脱离 GUI 在终端里运行的通用基础设施。

## 待回答的问题

- [x] Agent 架构
- [x] Tools 设计
- [x] Skills 机制
- [x] MCP 支持
- [x] 模型适配层如何抽象
- [x] 开源 Coding Agent 的可借鉴之处

### Agent 架构

OpenCode 的核心是「客户端/服务端分离」而非单体 CLI：

- **Server（核心运行时）**：承载 Agent Loop、会话状态、工具执行、权限判定、模型调用，对外暴露 HTTP/SDK 接口。所有「智能」都在这里，前端只是壳。
- **多前端客户端**：同一个 server 可被 TUI（Go + 终端 UI）、Web、IDE 插件、GitHub/GitLab 集成等多种客户端复用，因此可以本地跑、也可以把 server 跑在远程机器上再远程接入。
- **Agent Loop**：标准的「模型推理 → 产出 tool call → 执行工具 → 把结果回灌上下文 → 再推理」循环，直到任务完成或触发权限/中止。
- **Agents（子代理/角色）**：支持配置多种 agent（如主 agent 与专用 subagent），各自可限定可用工具、模型和权限，用于任务编排与分工。

这种架构的价值在于：UI 与推理解耦、便于远程/无头运行、便于把 OpenCode 当作可编程后端嵌入其它系统。

### Tools 设计

工具是 LLM 操作真实世界的唯一出口。OpenCode 的内置工具集与主流编码 Agent 高度一致：

| 工具 | 作用 |
| --- | --- |
| `read` / `write` / `edit` | 读文件、建/覆盖文件、精确字符串替换修改 |
| `apply_patch` | 以补丁/diff 形式批量增删改、重命名文件 |
| `grep` / `glob` | 基于 ripgrep 的内容正则搜索与文件名模式匹配（默认遵守 `.gitignore`） |
| `bash` | 执行任意 shell 命令（装依赖、跑测试、git 等） |
| `lsp`（实验性） | 调 LSP 做跳转定义、找引用、hover、调用层级等代码智能 |
| `webfetch` / `websearch` | 抓取指定 URL / 联网检索 |
| `skill` | 加载某个 `SKILL.md` 技能并注入上下文 |
| `question` | 执行过程中反问用户、让其在选项间选择 |

设计要点：

- **编辑以 `edit`（精确替换）为主**，而非让模型重写整个文件，减少 token 消耗和误改。
- **统一的权限模型**：每个工具可配 `allow` / `ask` / `deny`，支持通配符（如 `mymcp_*: ask`），把「危险操作需人工确认」做成一等公民。
- **可扩展**：除内置工具外，还能通过 Custom Tools（配置里定义函数）和 MCP server 挂载外部能力。

### Skills 机制

Skills 是「按需加载的任务专用说明书」，对应仓库里的 `SKILL.md` 文件：

- 平时技能只在目录中以「名称 + 简介」被登记，不占满上下文；当任务匹配到某技能时，模型调用 `skill` 工具把该 `SKILL.md` 的完整内容注入对话再执行。
- 这是一种**渐进式上下文加载（progressive disclosure）**：避免把所有领域知识一次性塞进 system prompt，既省 token 又让专长可复用、可版本化管理。
- 与 Anthropic 的 Agent Skills 理念一致，可用于封装「如何处理某类文件 / 遵循某套规范 / 调某条内部流水线」等重复性专家知识。

### MCP 支持

OpenCode 原生支持 Model Context Protocol：

- 可配置 **本地（stdio）** 和 **远程（HTTP/SSE）** 两类 MCP server，把数据库访问、API 集成、第三方服务等外部工具接入 Agent。
- MCP 暴露的工具与内置工具共享同一套权限体系，可用通配符统一要求审批（`"mymcp_*": "ask"`）。
- 意义：MCP 让「给 Agent 加能力」从改源码变成改配置，形成可插拔的工具生态；OpenCode 自身的 websearch 等能力也通过托管 MCP 服务提供。

### 模型适配层如何抽象

OpenCode 的核心卖点就是「不绑模型」：

- 通过一个统一的 **Providers / Models 抽象层**对接 Anthropic、OpenAI、Google、以及各类本地/第三方（OpenRouter、本地推理等）模型，用户在配置里切换 provider 和 model 即可，无需改业务逻辑。
- 模型元数据（上下文窗口、价格、能力、工具调用格式等）依赖 **models.dev** 这类开放模型数据库来描述，适配层据此拉平不同厂商在 API 协议、tool-calling 规范上的差异。
- 这样 Agent Loop 和工具层对「底层是哪家模型」保持无感，便于用新模型、做成本/质量 A-B、或在离线环境换本地模型。

### 开源 Coding Agent 的可借鉴之处

- **UI 与推理解耦（client/server）**：让同一引擎服务终端、Web、IDE、CI，便于远程/无头部署，值得在自研 Agent 中借鉴。
- **权限即一等公民**：per-tool 的 allow/ask/deny + 通配符，是「自主性」与「安全可控」的务实平衡。
- **工具集最小而完备**：read/edit/bash/grep/glob 这套「文件系统 + 执行 + 搜索」几乎是编码 Agent 的事实标准。
- **Skills 的渐进式上下文**：用按需加载替代臃肿 system prompt，是上下文工程的好范式。
- **模型适配 + MCP 两层可插拔**：模型侧不锁厂商、能力侧靠 MCP 扩展，是构建长期可维护 Agent 平台的关键解耦。

## 核心内容

- **定位**：开源、跨厂商、终端优先的 AI 编码 Agent，可自托管、可编程、可脱离图形界面运行。
- **架构**：core server（推理 + 工具 + 权限 + 会话）+ 多客户端（TUI/Web/IDE/GitHub 等），通过 SDK/HTTP 通信。
- **执行闭环**：Agent Loop 驱动「推理 → tool call → 执行 → 回灌 → 再推理」。
- **工具层**：内置 read/write/edit/apply_patch/grep/glob/bash/lsp/skill/webfetch/websearch/question；可用 Custom Tools 与 MCP 扩展；grep/glob 基于 ripgrep。
- **安全**：统一权限模型（allow/ask/deny + 通配符），危险操作可强制人工确认。
- **扩展机制**：Agents（角色/子代理）、Rules（AGENTS.md 等规则）、Skills（SKILL.md）、Plugins、Custom Tools、MCP、LSP。
- **模型层**：Providers/Models 抽象 + models.dev 元数据，任意切换模型。

## 与其他知识的关系

| 关系 | 知识点 | 说明 |
| --- | --- | --- |
| 上游 | Agent Loop | OpenCode 的运行内核就是标准的「推理-工具调用-回灌」循环，没有它就谈不上自主编码 |
| 上游 | Model Context Protocol (MCP) | 外部工具能力通过 MCP 标准接入，是其可扩展性的协议基础 |
| 下游 | 工程 CI/CD 与远程开发工作流 | client/server 架构使其可作为无头后端嵌入流水线、远程机器、GitHub/GitLab 自动化中 |
| 易混淆 | Claude Code / Cursor / Copilot | 同为编码 Agent，但它们闭源且绑定特定模型/编辑器；OpenCode 开源、provider-agnostic、终端优先 |

## 典型工程场景

- 在纯终端/SSH 远程服务器上让 Agent 直接改代码、跑测试、提交 PR，无需图形编辑器。
- 需要在不同模型间自由切换（成本/质量权衡、或切本地私有模型以满足数据合规）的团队。
- 把编码 Agent 作为可编程后端，嵌入 CI/CD、GitHub/GitLab 自动化（如自动修复、自动回复 issue）。
- 需要自托管、可审计、可二次开发的企业内部编码助手，而非把代码交给闭源 SaaS。
- 用 Skills/Rules/MCP 把团队内部规范、流水线、私有服务封装成 Agent 可复用的专家能力。

## 适用边界

**适合使用**

- 偏好终端工作流、或要在远程/无头环境运行编码 Agent。
- 需要不绑定单一模型厂商、要能自由切换/自托管模型。
- 需要开源、可审计、可二次开发与深度定制（自定义工具、插件、MCP）。
- 希望用统一权限模型控制 Agent 自主度、兼顾安全。

**不适合使用**

- 团队更看重开箱即用的图形 IDE 深度集成（内联补全、可视化 diff 等），GUI 优先的 Cursor 类产品体验更顺。
- 用户完全不熟悉命令行、不愿配置 provider/API key。
- 只需要轻量代码补全而非「自主完成多步任务」的场景，重型 Agent 属于杀鸡用牛刀。

## 常见误区

- 以为它只是一个「终端里的聊天框」——实际它能自主读写文件、执行命令、跑完整多步任务。
- 以为 OpenCode 自带/绑定某个模型——它本身不含模型，必须接入某个 provider 的 API 或本地模型。
- 以为开源就等于默认安全——默认所有工具 `allow` 且可执行任意 bash，生产环境必须配置权限（ask/deny）。
- 把 Skills 当成插件/代码扩展——Skills 只是按需注入的 Markdown 说明，不等于 Custom Tools 或 MCP 这类真正加能力的机制。
- 认为 TUI 就是全部——它只是接入 core server 的一个客户端，推理与工具执行都在 server 侧。

## 我的结论

> 当你需要一个「开源、可换任意模型、能在终端/远程无头运行、可深度定制」的自主编码 Agent 时，OpenCode 是当前最值得选的方案之一，尤其适合重视数据合规、要自托管、或想把 Agent 嵌入自动化流水线的团队。它的 client/server 解耦、per-tool 权限模型、模型适配层 + MCP 两级可插拔，都是构建长期可维护 Agent 平台的优秀范式，值得在自研时借鉴。
> 反过来，如果团队主要在图形 IDE 里工作、追求内联补全和可视化交互的顺滑体验，或者成员不习惯命令行配置，那么 Cursor/Copilot 这类 GUI 优先的闭源产品体验更好；只做轻量补全时也不必上这类重型 Agent。一句话：要「自由、可控、可自托管的自主编码引擎」就用它；要「省心的图形化补全体验」就别用它。

## 待解决问题

- [ ] OpenCode core server 具体用什么语言/运行时实现（TypeScript/Bun 还是其它），其 Agent Loop 的上下文压缩与长会话管理策略细节如何？
- [ ] 其权限模型在面对 MCP 外部工具与 bash 任意命令时，沙箱隔离到什么粒度，能否满足企业级安全审计要求？
- [ ] 模型适配层对不同厂商 tool-calling / 结构化输出差异的拉平程度，在弱 function-calling 模型上的实际稳定性如何？

## 参考资料

- OpenCode 官方文档（opencode.ai/docs）——Tools、Permissions、Agents、Skills、MCP servers、Models、Providers、Server/SDK 等章节
- OpenCode 开源仓库（GitHub: anomalyco/opencode）
- Model Context Protocol 官方规范（modelcontextprotocol.io）
- models.dev —— 开放模型元数据数据库（provider/模型能力与定价）
- Anthropic《Agent Skills / Building effective agents》等关于工具使用与渐进式上下文的工程文章
