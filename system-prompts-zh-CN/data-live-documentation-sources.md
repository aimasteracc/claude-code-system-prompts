<!--
name: 'Data: Live documentation sources'
description: WebFetch URLs for fetching current Claude API and Agent SDK documentation from official sources
ccVersion: 2.1.97
-->
# 实时文档来源

本文件包含用于从 platform.claude.com 和代理 SDK 仓库获取最新信息的 WebFetch URL。当用户需要自缓存内容上次更新以来可能已变更的最新数据时，请使用这些 URL。

## 何时使用 WebFetch

- 用户明确询问"最新"或"当前"信息时
- 缓存数据似乎不正确时
- 用户询问缓存内容未涵盖的功能时
- 用户需要特定 API 详情或示例时

## Claude API 文档 URL

### 模型与定价

| 主题           | URL                                                                   | 提取提示                                                               |
| --------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 模型概览 | `https://platform.claude.com/docs/en/about-claude/models/overview.md` | "Extract current model IDs, context windows, and pricing for all Claude models" |
| 定价         | `https://platform.claude.com/docs/en/pricing.md`                      | "Extract current pricing per million tokens for input and output"               |

### 核心功能

| 主题             | URL                                                                          | 提取提示                                                                      |
| ----------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 扩展思考 | `https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md` | "Extract extended thinking parameters, budget_tokens requirements, and usage examples" |
| 自适应思考 | `https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md` | "Extract adaptive thinking setup, effort levels, and {{OPUS_NAME}} usage examples"         |
| Effort 参数  | `https://platform.claude.com/docs/en/build-with-claude/effort.md`            | "Extract effort levels, cost-quality tradeoffs, and interaction with thinking"        |
| 工具使用          | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md`  | "Extract tool definition schema, tool_choice options, and handling tool results"       |
| 流式传输         | `https://platform.claude.com/docs/en/build-with-claude/streaming.md`         | "Extract streaming event types, SDK examples, and best practices"                      |
| 提示缓存    | `https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md`    | "Extract cache_control usage, pricing benefits, and implementation examples"           |

### 媒体与文件

| 主题       | URL                                                                    | 提取提示                                                 |
| ----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 视觉      | `https://platform.claude.com/docs/en/build-with-claude/vision.md`      | "Extract supported image formats, size limits, and code examples" |
| PDF 支持 | `https://platform.claude.com/docs/en/build-with-claude/pdf-support.md` | "Extract PDF handling capabilities, limits, and examples"         |

### API 操作

| 主题            | URL                                                                         | 提取提示                                                                                       |
| ---------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| 批次处理 | `https://platform.claude.com/docs/en/build-with-claude/batch-processing.md` | "Extract batch API endpoints, request format, and polling for results"                                  |
| Files API        | `https://platform.claude.com/docs/en/build-with-claude/files.md`            | "Extract file upload, download, and referencing in messages, including supported types and beta header" |
| 词元计数   | `https://platform.claude.com/docs/en/build-with-claude/token-counting.md`   | "Extract token counting API usage and examples"                                                         |
| 速率限制      | `https://platform.claude.com/docs/en/api/rate-limits.md`                    | "Extract current rate limits by tier and model"                                                         |
| 错误           | `https://platform.claude.com/docs/en/api/errors.md`                         | "Extract HTTP error codes, meanings, and retry guidance"                                                |

### 工具

| 主题          | URL                                                                                    | 提取提示                                                                        |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 代码执行 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md` | "Extract code execution tool setup, file upload, container reuse, and response handling" |
| 计算机使用   | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use.md`        | "Extract computer use tool setup, capabilities, and implementation examples"             |
| Bash 工具      | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/bash-tool.md`           | "Extract bash tool schema, reference implementation, and security considerations"        |
| 文本编辑器    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool.md`    | "Extract text editor tool commands, schema, and reference implementation"                |
| 记忆工具    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`         | "Extract memory tool commands, directory structure, and implementation patterns"         |
| 工具搜索    | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool.md`    | "Extract tool search setup, when to use, and cache interaction"                          |
| 程序化工具调用 | `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling.md` | "Extract PTC setup, script execution model, and tool invocation from code"    |
| 技能         | `https://platform.claude.com/docs/en/agents-and-tools/skills.md`                       | "Extract skill folder structure, SKILL.md format, and loading behavior"                  |

### 高级功能

| 主题              | URL                                                                           | 提取提示                                                   |
| ------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------- |
| 结构化输出 | `https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md` | "Extract output_config.format usage and schema enforcement"                           |
| 压缩         | `https://platform.claude.com/docs/en/build-with-claude/compaction.md`         | "Extract compaction setup, trigger config, and streaming with compaction"             |
| 上下文编辑    | `https://platform.claude.com/docs/en/build-with-claude/context-editing.md`    | "Extract context editing thresholds, what gets cleared, and configuration"            |
| 引用          | `https://platform.claude.com/docs/en/build-with-claude/citations.md`          | "Extract citation format and implementation"        |
| 上下文窗口    | `https://platform.claude.com/docs/en/build-with-claude/context-windows.md`    | "Extract context window sizes and token management" |

### 托管代理

当缓存的 `shared/managed-agents-*.md` 概念文件或 `{lang}/managed-agents/README.md` 中没有覆盖某个托管代理的绑定、行为或线路级细节时，请使用以下 URL。

| 主题                 | URL                                                                              | 提取提示                                                                               |
| --------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 概览              | `https://platform.claude.com/docs/en/managed-agents/overview.md`                 | "Extract the high-level architecture and how agents/sessions/environments/vaults fit together" |
| 快速入门            | `https://platform.claude.com/docs/en/managed-agents/quickstart.md`               | "Extract the minimal end-to-end agent → environment → session → stream code path"              |
| 代理设置           | `https://platform.claude.com/docs/en/managed-agents/agent-setup.md`              | "Extract agent create/update/list-versions/archive lifecycle and parameters"                   |
| 定义结果       | `https://platform.claude.com/docs/en/managed-agents/define-outcomes.md`          | "Extract outcome definitions, evaluation hooks, and success criteria configuration"             |
| 会话              | `https://platform.claude.com/docs/en/managed-agents/sessions.md`                 | "Extract session lifecycle, status transitions, idle/terminated semantics, and resume rules"    |
| 环境          | `https://platform.claude.com/docs/en/managed-agents/environments.md`             | "Extract environment config (cloud/networking), management endpoints, and reuse model"          |
| 事件与流式传输  | `https://platform.claude.com/docs/en/managed-agents/events-and-streaming.md`     | "Extract event stream types, stream-first ordering, reconnect/dedupe, and steering patterns"    |
| 工具                 | `https://platform.claude.com/docs/en/managed-agents/tools.md`                    | "Extract built-in toolset, custom tool definitions, and tool result wire format"                |
| 文件                 | `https://platform.claude.com/docs/en/managed-agents/files.md`                    | "Extract file upload, mount paths, session resources, and listing/downloading session outputs"  |
| 权限策略   | `https://platform.claude.com/docs/en/managed-agents/permission-policies.md`      | "Extract permission policy types (allow/deny/confirm) and per-tool config"                     |
| 多代理           | `https://platform.claude.com/docs/en/managed-agents/multi-agent.md`              | "Extract multi-agent composition patterns, sub-agent invocation, and result handoff"            |
| 可观测性         | `https://platform.claude.com/docs/en/managed-agents/observability.md`            | "Extract logging, tracing, and usage telemetry exposed by managed agents"                       |
| GitHub                | `https://platform.claude.com/docs/en/managed-agents/github.md`                   | "Extract github_repository resource shape, multi-repo mounting, and token rotation"             |
| MCP 连接器         | `https://platform.claude.com/docs/en/managed-agents/mcp-connector.md`            | "Extract MCP server declaration on agents and vault-based credential injection at session"     |
| 密钥库                | `https://platform.claude.com/docs/en/managed-agents/vaults.md`                   | "Extract vault create, credential add/rotate, OAuth refresh shape, and archive"                 |
| 技能                | `https://platform.claude.com/docs/en/managed-agents/skills.md`                   | "Extract skill packaging and loading model for managed agents"                                  |
| 记忆                | `https://platform.claude.com/docs/en/managed-agents/memory.md`                   | "Extract memory resource shape, scoping, and lifecycle"                                         |
| 入门引导            | `https://platform.claude.com/docs/en/managed-agents/onboarding.md`               | "Extract first-run setup, prerequisites, and account/region requirements"                      |
| 云容器      | `https://platform.claude.com/docs/en/managed-agents/cloud-containers.md`         | "Extract cloud container runtime, image config, and network/storage knobs"                     |
| 迁移             | `https://platform.claude.com/docs/en/managed-agents/migration.md`                | "Extract migration paths from earlier APIs/preview shapes to GA managed agents"                 |

### Anthropic CLI

`ant` CLI 提供对 Claude API 的终端访问。每个 API 资源都作为子命令公开。它是从版本控制的 YAML 创建代理、环境、会话和其他资源，以及交互式检查响应的便捷方式。

| 主题         | URL                                                     | 提取提示                                                                                  |
| ------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Anthropic CLI | `https://platform.claude.com/docs/en/api/sdks/cli.md`   | "Extract CLI install, authentication, command structure, and the beta:agents/environments/sessions commands" |

---

## Claude API SDK 仓库

当某个绑定（类、方法、命名空间、字段）未在缓存的 `{lang}/` 技能文件或上述托管代理文档中覆盖时，使用 WebFetch 获取这些仓库。SDK 包含对 `/v1/agents`、`/v1/sessions`、`/v1/environments` 及相关资源的 beta 托管代理支持——在仓库中搜索 `BetaManagedAgents`、`beta.agents`、`beta.sessions` 或该语言的等效命名空间。

| SDK        | URL                                                      | 提取提示                                                                                                       |
| ---------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Python     | `https://github.com/anthropics/anthropic-sdk-python`     | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| TypeScript | `https://github.com/anthropics/anthropic-sdk-typescript` | "Extract beta managed-agents namespaces, classes, and method signatures (`client.beta.agents`, `client.beta.sessions`)" |
| Java       | `https://github.com/anthropics/anthropic-sdk-java`       | "Extract beta managed-agents classes, builders, and method signatures (`client.beta().agents()`, `BetaManagedAgents*`)" |
| Go         | `https://github.com/anthropics/anthropic-sdk-go`         | "Extract beta managed-agents types and method signatures (`client.Beta.Agents`, `BetaManagedAgents*` event types)"      |
| Ruby       | `https://github.com/anthropics/anthropic-sdk-ruby`       | "Extract beta managed-agents methods and parameter shapes (`client.beta.agents`, `client.beta.sessions`)"               |
| C#         | `https://github.com/anthropics/anthropic-sdk-csharp`     | "Extract beta managed-agents classes and method signatures (NuGet package, `BetaManagedAgents*` types)"                 |
| PHP        | `https://github.com/anthropics/anthropic-sdk-php`        | "Extract beta managed-agents classes and method signatures (`$client->beta->agents`, `BetaManagedAgents*` params)"      |

---

## 回退策略

当 WebFetch 失败时（网络问题、URL 已变更）：

1. 使用特定语言文件中的缓存内容（注意缓存日期）
2. 告知用户数据可能已过时
3. 建议用户直接访问 platform.claude.com 或 GitHub 仓库
