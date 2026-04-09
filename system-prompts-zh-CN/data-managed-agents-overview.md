<!--
name: 'Data: Managed Agents overview'
description: Provides the agent with a comprehensive overview of the Managed Agents API architecture, mandatory agent-then-session flow, beta headers, documentation reading guide, and common pitfalls
ccVersion: 2.1.97
-->
# 托管代理 — 概述

托管代理为每个会话分配一个容器作为代理的工作空间。代理循环运行在 Anthropic 的编排层上；容器是代理*工具*实际执行的地方——bash 命令、文件操作、代码。您先创建一个持久化的**代理**配置（模型、系统提示词、工具、MCP 服务器、技能），然后启动引用该配置的**会话**。会话以流式方式将事件返回给您；您向其发送用户消息和工具结果。

## ⚠️ 强制流程：先创建代理（一次），再创建会话（每次运行）

**代理作为独立对象的原因：版本管理。** 代理是一个持久化的版本化配置——每次更新都会创建一个新的不可变版本，会话在创建时固定到某个版本。这让您可以迭代代理（调整提示词、添加工具），而不会中断已在运行的会话，在变更导致退步时可以回滚，并能并排 A/B 测试不同版本。如果每次运行都调用 `agents.create()`，以上功能都将无法实现。

每个会话都引用一个预先创建的 `/v1/agents` 对象。只需创建一次代理，保存其 ID，并在所有后续运行中复用。

| 步骤 | 调用 | 频率 |
|---|---|---|
| 1 | `POST /v1/agents` — `model`、`system`、`tools`、`mcp_servers`、`skills` 在此定义 | **仅一次。** 保存 `agent.id` **和** `agent.version`。 |
| 2 | `POST /v1/sessions` — `agent: "agent_abc123"` 或 `{type: "agent", id, version}` | **每次运行。** 字符串简写使用最新版本。 |

如果您正要在会话体上写 `sessions.create()`，并带有 `model`、`system` 或 `tools` 字段——**停下来**。这些字段属于 `agents.create()`。会话只接受一个*指针*。

**生成代码时，请将初始化与运行时分离。** `agents.create()` 应放在安装脚本中（或用 `if agent_id is None:` 守护的块中），而不是热路径的顶部。如果用户的代码在每次调用时都调用 `agents.create()`，则会积累孤立的代理对象，并在每次调用时承担创建延迟。正确的模式是：创建一次 → 持久化 ID（配置文件、环境变量、密钥管理器）→ 每次运行时加载 ID 并调用 `sessions.create()`。

**要更改代理的行为，请使用 `POST /v1/agents/{id}` — 不要创建新代理。** 每次更新都会增加版本号；正在运行的会话保留其固定版本，新会话获取最新版本（或通过 `{type: "agent", id, version}` 显式固定）。详见 `shared/managed-agents-core.md` → 代理 → 版本管理。

## Beta 请求头

托管代理目前处于 Beta 阶段。SDK 会自动设置所需的 Beta 请求头：

| Beta 请求头 | 启用的功能 |
| ------------------------------ | ---------------------------------------------------- |
| `managed-agents-2026-04-01`    | 代理、环境、会话、事件、会话资源、保险库、凭据 |
| `skills-2025-10-02`            | 技能 API（用于管理自定义技能定义）   |
| `files-api-2025-04-14`         | 文件 API，用于文件上传                           |

**注意：不要混用 Beta 请求头** — 如果您需要通过技能 API 或文件 API 上传技能或文件，需要使用上面列出的相应 Beta 请求头。但在使用第 1 行列出的任何托管代理端点时，**不需要**同时包含技能或文件 Beta 请求头。切勿混用 Beta 请求头，使用这些特定端点时应优先使用技能或文件 Beta 请求头。


## 阅读指南

| 用户想要...                       | 阅读这些文件                                        |
| -------------------------------------- | ------------------------------------------------------- |
| **从零开始 / "帮我搭建一个代理"** | `shared/managed-agents-onboarding.md` — 引导式访谈（位置→受众→功能→观察），然后生成代码 |
| 理解 API 的工作原理           | `shared/managed-agents-core.md`                         |
| 查看完整的端点参考        | `shared/managed-agents-api-reference.md`                |
| **创建代理**（必须的第一步） | `shared/managed-agents-core.md`（代理部分）+ 语言文件 |
| 更新/版本管理代理                | `shared/managed-agents-core.md`（代理 → 版本管理）— 更新，不要重新创建 |
| 创建会话                       | `shared/managed-agents-core.md` + `{lang}/managed-agents/README.md` |
| 配置工具和权限        | `shared/managed-agents-tools.md`                        |
| 设置 MCP 服务器                     | `shared/managed-agents-tools.md`（MCP 服务器部分）  |
| 流式传输事件 / 处理 tool_use        | `shared/managed-agents-events.md` + 语言文件       |
| 设置环境                    | `shared/managed-agents-environments.md` + 语言文件 |
| 上传文件 / 挂载仓库            | `shared/managed-agents-environments.md`（资源部分）     |
| 存储 MCP 凭据                  | `shared/managed-agents-tools.md`（保险库部分）       |

## 常见陷阱

- **先创建代理，再创建会话——无例外** — 会话的 `agent` 字段只接受字符串 ID 或 `{type: "agent", id, version}` 对象。`model`、`system`、`tools`、`mcp_servers`、`skills` 是 **`POST /v1/agents` 的顶层字段**，绝不放在 `sessions.create()` 上。如果用户还没有创建代理，那是每个示例的第零步。
- **代理只创建一次，而非每次运行** — `agents.create()` 是初始化步骤。保存返回的 `agent_id` 并复用；不要在热路径顶部调用 `agents.create()`。如果代理的配置需要更改，使用 `POST /v1/agents/{id}` — 每次更新都会创建一个新版本，会话可以固定到特定版本以确保可重现性。
- **MCP 认证通过保险库进行** — 代理的 `mcp_servers` 数组只声明 `{type, name, url}`（无认证）。凭据存放在保险库中（`client.beta.vaults.credentials.create`），并通过 `vault_ids` 附加到会话。Anthropic 使用存储的刷新令牌自动刷新 OAuth 令牌。
- **通过流式传输获取事件** — `GET /v1/sessions/{id}/events/stream` 是实时接收代理输出的主要方式。
- **SSE 流不支持重播——断线后需合并重连** — 如果流在 `agent.tool_use`、`agent.mcp_tool_use` 或 `agent.custom_tool_use` 等待解决时（前两种等 `user.tool_confirmation`，最后一种等 `user.custom_tool_result`）中断，会话将死锁（客户端断开 → 会话空闲 → 重连 → 客户端不再处理解决事件）。每次（重）连接时：用 `GET /v1/sessions/{id}/events/stream` 打开流，获取 `GET /v1/sessions/{id}/events`，按事件 ID 去重，然后继续。详见 `shared/managed-agents-events.md` → 流中断后重连。
- **不要将 HTTP 库超时视为挂钟上限** — `requests` 的 `timeout=(c, r)` 和 `httpx.Timeout(n)` 是*每块*读取超时，每来一个字节就会重置，因此涓流连接可能无限期阻塞。如需对原始 HTTP 轮询设置硬性截止时间，在循环层面追踪 `time.monotonic()` 并显式中断。优先使用 SDK 的 `sessions.events.stream()` / `session.events.list()`，而非手写 HTTP。详见 `shared/managed-agents-events.md` → 接收事件。
- **消息排队** — 您可以在会话处于 `running` 或 `idle` 状态时发送事件；它们按顺序处理。无需等待响应后再发送下一条消息。
- **仅支持云环境** — `config.type: "cloud"` 是唯一受支持的环境类型。
