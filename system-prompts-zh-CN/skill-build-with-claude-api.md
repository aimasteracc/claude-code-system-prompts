<!--
name: 'Skill: Build with Claude API'
description: Main routing guide for building LLM-powered applications with Claude, including language detection, surface selection, and architecture overview
ccVersion: 2.1.91
-->
# 使用 Claude 构建 LLM 驱动的应用

此技能帮助你使用 Claude 构建 LLM 驱动的应用。根据需求选择合适的接口，检测项目语言，然后阅读相关的语言特定文档。

## 默认设置

除非用户另有要求：

对于 Claude 模型版本，请使用 {{OPUS_NAME}}，可通过精确模型字符串 `{{OPUS_ID}}` 访问。对于任何稍微复杂的内容，请默认使用自适应思考（`thinking: {type: "adaptive"}`）。最后，对于任何可能涉及长输入、长输出或高 `max_tokens` 的请求，请默认使用流式传输——它可以防止达到请求超时。如果不需要处理单个流事件，使用 SDK 的 `.get_final_message()` / `.finalMessage()` 帮助方法获取完整响应。

---

## 语言检测

在阅读代码示例之前，确定用户使用的语言：

1. **查看项目文件**以推断语言：

   - `*.py`、`requirements.txt`、`pyproject.toml`、`setup.py`、`Pipfile` → **Python** — 从 `python/` 读取
   - `*.ts`、`*.tsx`、`package.json`、`tsconfig.json` → **TypeScript** — 从 `typescript/` 读取
   - `*.js`、`*.jsx`（无 `.ts` 文件） → **TypeScript** — JS 使用相同的 SDK，从 `typescript/` 读取
   - `*.java`、`pom.xml`、`build.gradle` → **Java** — 从 `java/` 读取
   - `*.kt`、`*.kts`、`build.gradle.kts` → **Java** — Kotlin 使用 Java SDK，从 `java/` 读取
   - `*.scala`、`build.sbt` → **Java** — Scala 使用 Java SDK，从 `java/` 读取
   - `*.go`、`go.mod` → **Go** — 从 `go/` 读取
   - `*.rb`、`Gemfile` → **Ruby** — 从 `ruby/` 读取
   - `*.cs`、`*.csproj` → **C#** — 从 `csharp/` 读取
   - `*.php`、`composer.json` → **PHP** — 从 `php/` 读取

2. **检测到多种语言**（例如同时有 Python 和 TypeScript 文件）：

   - 检查用户当前文件或问题与哪种语言相关
   - 如果仍不明确，询问："我检测到 Python 和 TypeScript 文件。你在哪种语言中集成 Claude API？"

3. **无法推断语言**（空项目、无源文件或不支持的语言）：

   - 使用 AskUserQuestion，选项为：Python、TypeScript、Java、Go、Ruby、cURL/原始 HTTP、C#、PHP
   - 如果 AskUserQuestion 不可用，默认使用 Python 示例并注明："显示 Python 示例。如需其他语言请告知。"

4. **检测到不支持的语言**（Rust、Swift、C++、Elixir 等）：

   - 建议使用 `curl/` 中的 cURL/原始 HTTP 示例，并注明可能存在社区 SDK
   - 提供显示 Python 或 TypeScript 示例作为参考实现

5. **如果用户需要 cURL/原始 HTTP 示例**，从 `curl/` 读取。

### 语言特定功能支持

| 语言 | 工具运行器 | 代理 SDK | 说明 |
| --- | --- | --- | --- |
| Python | 是（beta） | 是 | 全面支持——`@beta_tool` 装饰器 |
| TypeScript | 是（beta） | 是 | 全面支持——`betaZodTool` + Zod |
| Java | 是（beta） | 否 | 带注解类的 beta 工具使用 |
| Go | 是（beta） | 否 | `toolrunner` 包中的 `BetaToolRunner` |
| Ruby | 是（beta） | 否 | beta 版的 `BaseTool` + `tool_runner` |
| cURL | 不适用 | 不适用 | 原始 HTTP，无 SDK 功能 |
| C# | 否 | 否 | 官方 SDK |
| PHP | 是（beta） | 否 | `BetaRunnableTool` + `toolRunner()` |

---

## 我应该使用哪个接口？

> **从简单开始。** 默认使用满足你需求的最简单层级。单次 API 调用和工作流可以处理大多数用例——只有当任务确实需要开放式的、模型驱动的探索时，才考虑代理。

| 用例 | 层级 | 推荐接口 | 原因 |
| --- | --- | --- | --- |
| 分类、摘要、提取、问答 | 单次 LLM 调用 | **Claude API** | 一次请求，一次响应 |
| 批处理或嵌入 | 单次 LLM 调用 | **Claude API** | 专用端点 |
| 代码控制逻辑的多步骤流水线 | 工作流 | **Claude API + 工具使用** | 你控制循环 |
| 使用自定义工具的自定义代理 | 代理 | **Claude API + 工具使用** | 最大灵活性 |
| 具有文件/网络/终端访问的 AI 代理 | 代理 | **代理 SDK** | 内置工具、安全性和 MCP 支持 |
| 代码辅助助手 | 代理 | **代理 SDK** | 专为此用例设计 |
| 需要内置权限和护栏 | 代理 | **代理 SDK** | 包含安全功能 |

> **注意：** 代理 SDK 适用于你想要内置文件/网络/终端工具、权限和开箱即用的 MCP 的情况。如果你想构建一个使用自定义工具的代理，Claude API 是正确的选择——使用工具运行器进行自动循环处理，或使用手动循环进行细粒度控制（审批门控、自定义日志记录、条件执行）。

### 决策树

```
你的应用需要什么？

1. 单次 LLM 调用（分类、摘要、提取、问答）
   └── Claude API — 一次请求，一次响应

2. Claude 是否需要读写文件、浏览网络或运行 shell 命令
   作为其工作的一部分？（不是：你的应用读取文件并传给 Claude——
   Claude 本身是否需要发现和访问文件/网络/shell？）
   └── 是 → 代理 SDK — 内置工具，不要重新实现它们
       示例："扫描代码库查找 bug"、"摘要目录中的每个文件"、
             "使用子代理查找 bug"、"通过网络搜索研究话题"

3. 工作流（多步骤、代码编排、自定义工具）
   └── Claude API + 工具使用 — 你控制循环

4. 开放式代理（模型决定自己的轨迹，自定义工具）
   └── Claude API 代理循环（最大灵活性）
```

### 我应该构建代理吗？

在选择代理层级之前，检查所有四个标准：

- **复杂性** — 任务是否是多步骤且难以提前完全规定的？（例如"将这份设计文档变成 PR"vs"从这个 PDF 中提取标题"）
- **价值** — 结果是否能证明更高的成本和延迟是合理的？
- **可行性** — Claude 是否擅长这种任务类型？
- **出错成本** — 错误能被发现并从中恢复吗？（测试、审查、回滚）

如果其中任何一个答案是"否"，请停留在更简单的层级（单次调用或工作流）。

---

## 架构

一切都通过 `POST /v1/messages` 进行。工具和输出约束是这个单一端点的功能——不是独立的 API。

**用户定义的工具** — 你定义工具（通过装饰器、Zod 模式或原始 JSON），SDK 的工具运行器处理调用 API、执行你的函数，并循环直到 Claude 完成。对于完全控制，你可以手动编写循环。

**服务器端工具** — 在 Anthropic 基础设施上运行的 Anthropic 托管工具。代码执行是完全服务器端的（在 `tools` 中声明，Claude 自动运行代码）。Computer use 可以是服务器托管或自托管。

**结构化输出** — 限制 Messages API 响应格式（`output_config.format`）和/或工具参数验证（`strict: true`）。推荐方法是 `client.messages.parse()`，它会自动根据你的模式验证响应。注意：旧的 `output_format` 参数已废弃；在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**支持端点** — Batches（`POST /v1/messages/batches`）、Files（`POST /v1/files`）、Token Counting 和 Models（`GET /v1/models`、`GET /v1/models/{id}` — 实时能力/上下文窗口发现）支持或补充 Messages API 请求。

---

## 当前模型（缓存于：2026-02-17）

| 模型 | 模型 ID | 上下文 | 输入 $/1M | 输出 $/1M |
| --- | --- | --- | --- | --- |
| Claude Opus 4.6 | `claude-opus-4-6` | 200K（1M beta） | $5.00 | $25.00 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K（1M beta） | $3.00 | $15.00 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1.00 | $5.00 |

**除非用户明确指定其他模型，否则始终使用 `{{OPUS_ID}}`。** 这是不可妥协的。不要使用 `{{SONNET_ID}}`、`{{PREV_SONNET_ID}}` 或任何其他模型，除非用户字面上说"用 sonnet"或"用 haiku"。不要以节省成本为由降级——那是用户的决定，不是你的。

**关键：只使用上表中精确的模型 ID 字符串——它们本身就是完整的。不要追加日期后缀。** 例如，使用 `claude-sonnet-4-5`，永远不要使用 `claude-sonnet-4-5-20250514` 或你可能从训练数据中记得的任何其他带日期后缀的变体。如果用户请求表中没有的旧模型（例如"opus 4.5"、"sonnet 3.7"），阅读 `shared/models.md` 获取精确 ID——不要自己构造。

说明：如果上面的任何模型字符串对你来说看起来陌生，这是预期的——这只意味着它们是在你的训练数据截止日期之后发布的。请放心，它们是真实的模型；我们不会这样捉弄你。

**实时能力查询：** 上表是缓存的。当用户询问"X 的上下文窗口是多少"、"X 是否支持视觉/思考/努力"或"哪些模型支持 Y"时，查询 Models API（`client.models.retrieve(id)` / `client.models.list()`）——字段参考和能力过滤示例见 `shared/models.md`。

---

## 思考与努力（快速参考）

**Opus 4.6 — 自适应思考（推荐）：** 使用 `thinking: {type: "adaptive"}`。Claude 动态决定何时以及思考多少。不需要 `budget_tokens`——`budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上已废弃，不得使用。自适应思考还自动启用交错思考（无需 beta 标头）。**当用户要求"扩展思考"、"思考预算"或 `budget_tokens` 时：始终使用 Opus 4.6 配合 `thinking: {type: "adaptive"}`。固定词元预算的概念已废弃——自适应思考取代了它。不要使用 `budget_tokens`，也不要切换到旧模型。**

**努力参数（GA，无需 beta 标头）：** 通过 `output_config: {effort: "low"|"medium"|"high"|"max"}` 控制思考深度和整体词元消耗（在 `output_config` 内，而非顶层）。默认为 `high`（等同于省略它）。`max` 仅限 Opus 4.6。适用于 Opus 4.5、Opus 4.6 和 Sonnet 4.6。在 Sonnet 4.5 / Haiku 4.5 上会报错。与自适应思考结合可获得最佳成本-质量权衡。较低的努力意味着更少且更合并的工具调用、更少的前言和更简短的确认——`medium` 通常是有利的平衡；当正确性比成本更重要时使用 `max`；对于子代理或简单任务使用 `low`。

**Sonnet 4.6：** 支持自适应思考（`thinking: {type: "adaptive"}`）。`budget_tokens` 在 Sonnet 4.6 上已废弃——改用自适应思考。

**旧版模型（仅在明确要求时）：** 如果用户特别要求 Sonnet 4.5 或其他旧版模型，使用 `thinking: {type: "enabled", budget_tokens: N}`。`budget_tokens` 必须小于 `max_tokens`（最小 1024）。不要因为用户提到 `budget_tokens` 就选择旧版模型——改用 Opus 4.6 配合自适应思考。

---

## 压缩（快速参考）

**Beta，Opus 4.6 和 Sonnet 4.6。** 对于可能超过 200K 上下文窗口的长时间对话，启用服务器端压缩。API 在接近触发阈值（默认：150K 词元）时自动摘要较早的上下文。需要 beta 标头 `compact-2026-01-12`。

**关键：** 每个轮次都将 `response.content`（而非只是文字）追加回你的消息中。响应中的压缩块必须被保留——API 使用它们在下一次请求时替换压缩的历史记录。只提取文字字符串并追加它将会静默丢失压缩状态。

代码示例见 `{lang}/claude-api/README.md`（压缩部分）。完整文档通过 WebFetch 在 `shared/live-sources.md` 中获取。

---

## 提示词缓存（快速参考）

**前缀匹配。** 前缀中任何位置的任何字节变化都会使其后的所有内容失效。渲染顺序为 `tools` → `system` → `messages`。将稳定内容放在前面（冻结的系统提示词、确定性工具列表），将易变内容（时间戳、每次请求的 ID、变化的问题）放在最后一个 `cache_control` 断点之后。

**顶层自动缓存**（`messages.create()` 上的 `cache_control: {type: "ephemeral"}`）是在不需要精细放置时最简单的选项。每次请求最多 4 个断点。最小可缓存前缀约为 1024 个词元——更短的前缀会静默不缓存。

**通过 `usage.cache_read_input_tokens` 验证** — 如果在重复请求中它为零，则有静默失效器在起作用（系统提示词中的 `datetime.now()`、未排序的 JSON、变化的工具集）。

关于放置模式、架构指导和静默失效审计清单：阅读 `shared/prompt-caching.md`。语言特定语法：`{lang}/claude-api/README.md`（提示词缓存部分）。

<!-- __S3__ -->

---

## 阅读指南

检测到语言后，根据用户需求阅读相关文件：

### 快速任务参考

**单次文本分类/摘要/提取/问答：**
→ 只阅读 `{lang}/claude-api/README.md`

**聊天 UI 或实时响应显示：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间对话（可能超过上下文窗口）：**
→ 阅读 `{lang}/claude-api/README.md`——见压缩部分

**提示词缓存/优化缓存/"为什么我的缓存命中率低"：**
→ 阅读 `shared/prompt-caching.md` + `{lang}/claude-api/README.md`（提示词缓存部分）

**函数调用/工具使用/代理：**
→ 阅读 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**代理设计（工具接口、上下文管理、缓存策略）：**
→ 阅读 `shared/agent-design.md`

**批处理（非延迟敏感）：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求上传文件：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**带内置工具的代理（文件/网络/终端）：**
→ 阅读 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

### Claude API（完整文件参考）

阅读**语言特定的 Claude API 文件夹**（`{language}/claude-api/`）：

1. **`{language}/claude-api/README.md`** — **首先阅读此文件。** 安装、快速开始、常见模式、错误处理。
2. **`shared/tool-use-concepts.md`** — 当用户需要函数调用、代码执行、记忆或结构化输出时阅读。涵盖概念基础。
3. **`shared/agent-design.md`** — 设计代理时阅读：bash vs 专用工具、编程式工具调用、工具搜索/技能、上下文编辑 vs 压缩 vs 记忆、缓存原则。
4. **`{language}/claude-api/tool-use.md`** — 阅读语言特定的工具使用代码示例（工具运行器、手动循环、代码执行、记忆、结构化输出）。
5. **`{language}/claude-api/streaming.md`** — 构建增量显示响应的聊天 UI 或界面时阅读。
6. **`{language}/claude-api/batches.md`** — 离线处理许多请求时阅读（非延迟敏感）。以 50% 的成本异步运行。
7. **`{language}/claude-api/files-api.md`** — 跨多个请求发送同一文件而无需重新上传时阅读。
8. **`shared/prompt-caching.md`** — 添加或优化提示词缓存时阅读。涵盖前缀稳定性设计、断点放置和静默使缓存失效的反模式。
9. **`shared/error-codes.md`** — 调试 HTTP 错误或实现错误处理时阅读。
10. **`shared/live-sources.md`** — 用于获取最新官方文档的 WebFetch URL。

> **注意：** 对于 Java、Go、Ruby、C#、PHP 和 cURL——这些每个只有一个文件，涵盖所有基础知识。阅读该文件以及按需阅读 `shared/tool-use-concepts.md` 和 `shared/error-codes.md`。

### 代理 SDK

阅读**语言特定的代理 SDK 文件夹**（`{language}/agent-sdk/`）。代理 SDK 仅适用于 **Python 和 TypeScript**。

1. **`{language}/agent-sdk/README.md`** — 安装、快速开始、内置工具、权限、MCP、钩子。
2. **`{language}/agent-sdk/patterns.md`** — 自定义工具、钩子、子代理、MCP 集成、会话恢复。
3. **`shared/live-sources.md`** — 获取当前代理 SDK 文档的 WebFetch URL。

---

## 何时使用 WebFetch

在以下情况下使用 WebFetch 获取最新文档：

- 用户询问"最新"或"当前"信息
- 缓存数据似乎不正确
- 用户询问此处未涵盖的功能

实时文档 URL 在 `shared/live-sources.md` 中。

## 常见陷阱

- 向 API 传递文件或内容时不要截断输入。如果内容太长无法放入上下文窗口，通知用户并讨论选项（分块、摘要等），而非静默截断。
- **Opus 4.6 / Sonnet 4.6 思考：** 使用 `thinking: {type: "adaptive"}`——不要使用 `budget_tokens`（两者都已废弃）。对于旧版模型，`budget_tokens` 必须小于 `max_tokens`（最小 1024）。错误时会抛出错误。
- **Opus 4.6 预填充已移除：** 助手消息预填充（最后助手轮次预填充）在 Opus 4.6 上返回 400 错误。改用结构化输出（`output_config.format`）或系统提示词指令来控制响应格式。
- **`max_tokens` 默认值：** 不要把 `max_tokens` 设得太低——达到上限会在思考中截断输出并需要重试。对于非流式传输请求，默认约为 `~16000`（在 SDK HTTP 超时下保持响应）。对于流式传输请求，默认约为 `~64000`（超时不是问题，所以给模型更多空间）。只有在有充分理由时才降低：分类（`~256`）、成本上限或刻意短输出。
- **128K 输出词元：** Opus 4.6 支持最多 128K `max_tokens`，但 SDK 在值那么大时需要流式传输以避免 HTTP 超时。使用 `.stream()` 配合 `.get_final_message()` / `.finalMessage()`。
- **工具调用 JSON 解析（Opus 4.6）：** Opus 4.6 可能在工具调用 `input` 字段中产生不同的 JSON 字符串转义（例如 Unicode 或正斜杠转义）。始终用 `json.loads()` / `JSON.parse()` 解析工具输入——永远不要对序列化的输入进行原始字符串匹配。
- **结构化输出（所有模型）：** 在 `messages.create()` 上使用 `output_config: {format: {...}}` 代替已废弃的 `output_format` 参数。这是一般 API 更改，不是 4.6 特有的。
- **不要重新实现 SDK 功能：** SDK 提供了高级辅助方法——使用它们而非从头构建。具体来说：使用 `stream.finalMessage()` 而非将 `.on()` 事件包装在 `new Promise()` 中；使用类型异常类（`Anthropic.RateLimitError` 等）而非字符串匹配错误消息；使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而非重新定义等价接口。
- **不要为 SDK 数据结构定义自定义类型：** SDK 导出所有 API 对象的类型。对消息使用 `Anthropic.MessageParam`，对工具定义使用 `Anthropic.Tool`，对工具结果使用 `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam`，对响应使用 `Anthropic.Message`。定义自己的 `interface ChatMessage { role: string; content: unknown }` 会复制 SDK 已提供的内容并失去类型安全性。
- **报告和文档输出：** 对于生成报告、文档或可视化的任务，代码执行沙箱预装了 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 和 `pypdf`。Claude 可以生成格式化文件（DOCX、PDF、图表）并通过 Files API 返回——对于"报告"或"文档"类型的请求，考虑这种方式而非纯文本 stdout。
