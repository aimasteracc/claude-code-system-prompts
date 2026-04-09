<!--
name: 'Data: Live documentation sources'
description: WebFetch URLs for fetching current Claude API and Agent SDK documentation from official sources
ccVersion: 2.1.91
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
| --------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------- |
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

---

## Claude API SDK 仓库

| SDK        | URL                                                       | 描述                    |
| ---------- | --------------------------------------------------------- | ------------------------------ |
| Python     | `https://github.com/anthropics/anthropic-sdk-python`     | `anthropic` pip 包源码 |
| TypeScript | `https://github.com/anthropics/anthropic-sdk-typescript` | `@anthropic-ai/sdk` npm 包源码 |
| Java       | `https://github.com/anthropics/anthropic-sdk-java`       | `anthropic-java` Maven 包源码  |
| Go         | `https://github.com/anthropics/anthropic-sdk-go`         | Go 模块源码               |
| Ruby       | `https://github.com/anthropics/anthropic-sdk-ruby`       | `anthropic` gem 源码         |
| C#         | `https://github.com/anthropics/anthropic-sdk-csharp`     | NuGet 包源码           |
| PHP        | `https://github.com/anthropics/anthropic-sdk-php`        | Composer 包源码        |

---

## 代理 SDK 文档 URL

### 核心文档

| 主题                | URL                                                         | 提取提示                                               |
| -------------------- | ----------------------------------------------------------- | --------------------------------------------------------------- |
| 代理 SDK 概览   | `https://platform.claude.com/docs/en/agent-sdk.md`          | "Extract the Agent SDK overview, key features, and use cases"   |
| 代理 SDK Python     | `https://github.com/anthropics/claude-agent-sdk-python`     | "Extract Python SDK installation, imports, and basic usage"     |
| 代理 SDK TypeScript | `https://github.com/anthropics/claude-agent-sdk-typescript` | "Extract TypeScript SDK installation, imports, and basic usage" |

### SDK 参考（GitHub README）

| 主题          | URL                                                                                       | 提取提示                                                            |
| -------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Python SDK     | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-python/main/README.md`     | "Extract Python SDK API reference, classes, and methods"     |
| TypeScript SDK | `https://raw.githubusercontent.com/anthropics/claude-agent-sdk-typescript/main/README.md` | "Extract TypeScript SDK API reference, types, and functions" |

### npm/PyPI 包

| 包                             | URL                                                            | 描述               |
| ----------------------------------- | -------------------------------------------------------------- | ------------------------- |
| claude-agent-sdk (Python)           | `https://pypi.org/project/claude-agent-sdk/`                   | PyPI 上的 Python 包    |
| @anthropic-ai/claude-agent-sdk (TS) | `https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk` | npm 上的 TypeScript 包 |

### GitHub 仓库

| 资源       | URL                                                         | 描述                         |
| -------------- | ----------------------------------------------------------- | ----------------------------------- |
| Python SDK     | `https://github.com/anthropics/claude-agent-sdk-python`     | Python 包源码               |
| TypeScript SDK | `https://github.com/anthropics/claude-agent-sdk-typescript` | TypeScript/Node.js 包源码   |
| MCP 服务器    | `https://github.com/modelcontextprotocol`                   | 官方 MCP 服务器实现 |

---

## 回退策略

当 WebFetch 失败时（网络问题、URL 已变更）：

1. 使用特定语言文件中的缓存内容（注意缓存日期）
2. 告知用户数据可能已过时
3. 建议用户直接访问 platform.claude.com 或 GitHub 仓库
