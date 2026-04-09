<!--
name: 'Data: Managed Agents tools and skills'
description: Reference documentation covering the Managed Agents SDK's tool types (agent toolset, MCP, custom), permission policies, vault credential management, and skills API for building specialized agents
ccVersion: 2.1.97
-->
# 托管代理 — 工具与技能

## 工具

### 服务端工具与客户端工具

| 类型 | 谁执行 | 工作方式 |
|---|---|---|
| **预构建 Claude 代理工具**（`agent_toolset_20260401`） | Anthropic，在会话容器上 | 文件操作、bash、Web 搜索等。一次性全部启用，或通过 `enabled: true/false` 单独配置。 |
| **MCP 工具**（`mcp_toolset`） | Anthropic，在会话容器上 | 由已连接 MCP 服务器提供的能力。通过工具集按服务器授予访问权限。 |
| **自定义工具** | **您** — 您的应用程序处理调用并返回结果 | 代理发出 `agent.custom_tool_use` 事件，会话进入 `idle`，您发回 `user.custom_tool_result` 事件。 |

**建议：** 通过 `agent_toolset_20260401` 启用所有预构建工具，然后按需单独禁用。

**版本管理：** 工具集是版本化的静态资源。当底层工具发生变化时，会创建新的工具集版本（因此有 `_20260401`），这样您始终清楚地知道获得的是什么。

### 代理工具集

`agent_toolset_20260401` 提供以下内置工具：

| 工具                   | 说明                              |
| ---------------------- | ---------------------------------------- |
| `bash` | 在 Shell 会话中执行 bash 命令 |
| `read` | 从本地文件系统读取文件，包括文本、图像、PDF 和 Jupyter 笔记本 |
| `write` | 向本地文件系统写入文件 |
| `edit` | 在文件中执行字符串替换 |
| `glob` | 使用 glob 模式快速匹配文件 |
| `grep` | 使用正则表达式搜索文本 |
| `web_fetch` | 从 URL 获取内容 |
| `web_search` | 在 Web 上搜索信息 |

启用完整工具集：

```json
{
  "tools": [
    { "type": "agent_toolset_20260401" }
  ]
}
```

### 按工具配置

覆盖各个工具的默认设置。以下示例启用除 bash 之外的所有工具：

```json
{
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "enabled": true },
      "configs": [
        { "name": "bash", "enabled": false }
      ]
    }
  ]
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `type` | ✅ | `"agent_toolset_20260401"` |
| `default_config` | ❌ | 应用于所有工具。`{ "enabled": bool, "permission_policy": {...} }` |
| `configs` | ❌ | 按工具覆盖：`[{ "name": "...", "enabled": bool, "permission_policy": {...} }]` |

### 权限策略

控制服务端执行的工具（代理工具集 + MCP）何时自动运行，何时等待批准。不适用于自定义工具。

| 策略 | 行为 |
|---|---|
| `always_allow` | 工具自动执行（默认） |
| `always_ask` | 会话发出 `session.status_idle` 并暂停，直到您发送 `tool_confirmation` 事件 |

```json
{
  "type": "agent_toolset_20260401",
  "default_config": {
    "enabled": true,
    "permission_policy": { "type": "always_allow" }
  },
  "configs": [
    { "name": "bash", "permission_policy": { "type": "always_ask" } }
  ]
}
```

**响应 `always_ask`：** 发送 `user.tool_confirmation` 事件，附带触发事件中 `agent_tool_use`/`mcp_tool_use` 的 `tool_use_id`：

```json
{ "type": "tool_confirmation", "tool_use_id": "sevt_abc123", "result": "allow" }
{ "type": "tool_confirmation", "tool_use_id": "sevt_def456", "result": "deny", "message": "Read .env.example instead" }
```

拒绝时的可选 `message` 会传递给代理，以便它调整方式。

要只启用特定工具，将默认设为关闭并逐个工具选择启用：

```json
{
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "enabled": false },
      "configs": [
        { "name": "bash", "enabled": true },
        { "name": "read", "enabled": true }
      ]
    }
  ]
}
```

### 自定义工具（客户端）

自定义工具由**您的应用程序**执行，而非 Anthropic。流程：

1. 代理决定使用该工具 → 会话发出携带输入的 `agent.custom_tool_use` 事件
2. 会话进入 `idle`，等待您
3. 您的应用程序执行该工具
4. 您发回携带输出的 `user.custom_tool_result` 事件
5. 会话恢复 `running`

无需权限策略——您是执行者。

```json
{
  "tools": [
    {
      "type": "custom",
      "name": "get_weather",
      "description": "Fetch current weather for a city.",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": { "type": "string", "description": "City name" }
        },
        "required": ["city"]
      }
    }
  ]
}
```

### MCP 服务器

MCP（模型上下文协议）服务器提供标准化的第三方能力（如 Asana、GitHub、Linear）。**配置分为代理和保险库两部分：**

1. **代理创建**时声明要连接的服务器（`type`、`name`、`url` — 无认证）。代理的 `mcp_servers` 数组没有认证字段。
2. **保险库**存储 OAuth 凭据。通过会话创建时的 `vault_ids` 附加。

这使密钥不出现在可复用的代理定义中。每个保险库凭据绑定到一个 MCP 服务器 URL；Anthropic 通过 URL 匹配凭据和服务器。

**代理端 — 声明服务器（无认证）：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `type` | ✅ | `"url"` |
| `name` | ✅ | 唯一名称——由 `mcp_toolset.mcp_server_name` 引用 |
| `url` | ✅ | MCP 服务器的端点 URL（Streamable HTTP 传输） |

```json
{
  "mcp_servers": [
    { "type": "url", "name": "linear", "url": "https://mcp.linear.app/mcp" }
  ],
  "tools": [
    { "type": "mcp_toolset", "mcp_server_name": "linear" }
  ]
}
```

**会话端 — 附加保险库：**

```json
{
  "agent": "agent_abc123",
  "environment_id": "env_abc123",
  "vault_ids": ["vlt_abc123"]
}
```

> 💡 **按工具启用（经验观察）：** `mcp_toolset` 已被观察到接受 `default_config: {enabled: false}` + `configs: [{name, enabled: true}]` 用于白名单模式。API 参考只展示了最简单的 `{type, mcp_server_name}` 形式。

> ⚠️ **MCP 认证令牌 ≠ REST API 令牌。** 托管 MCP 服务器（`mcp.notion.com`、`mcp.linear.app` 等）通常需要 **OAuth Bearer 令牌**，而非服务的原生 API 密钥。Notion 的 `ntn_` 集成令牌用于向 Notion 的 REST API 认证，但**不能**作为 Notion MCP 服务器的保险库凭据使用。这是两个不同的认证系统。

### 保险库 — MCP 凭据存储

**保险库**存储 OAuth 凭据（访问令牌 + 刷新令牌），Anthropic 通过标准 OAuth 2.0 `refresh_token` 授权自动代您刷新。这是在发布 SDK 中认证 MCP 服务器的唯一方式。

> 内部以前称为 TAT（工具/租户访问令牌）。

**流程：**

1. 创建保险库（`client.beta.vaults.create(...)`）——每个租户/用户一个，或共享一个，取决于您的模型
2. 向其添加 MCP 凭据（`client.beta.vaults.credentials.create(...)`）——每个凭据绑定一个 MCP 服务器 URL
3. 通过 `vault_ids: ["vlt_..."]` 在会话创建时引用保险库
4. Anthropic 在令牌过期前自动刷新；代理调用 MCP 工具时使用当前访问令牌

**凭据结构**：

```json
{
  "display_name": "Notion (workspace-foo)",
  "auth": {
    "type": "mcp_oauth",
    "mcp_server_url": "https://mcp.notion.com/mcp",
    "access_token": "<current access token>",
    "expires_at": "2026-04-02T14:00:00Z",
    "refresh": {
      "refresh_token": "<refresh token>",
      "client_id": "<your OAuth client_id>",
      "token_endpoint": "https://api.notion.com/v1/oauth/token",
      "token_endpoint_auth": { "type": "none" }
    }
  }
}
```

`refresh` 块是实现自动刷新的关键——`token_endpoint` 是 Anthropic 发送 `refresh_token` 授权请求的地址。`token_endpoint_auth` 是一个联合类型：

| `type` | 结构 | 使用场景 |
|---|---|---|
| `"none"` | `{type: "none"}` | 公开 OAuth 客户端（无密钥） |
| `"client_secret_basic"` | `{type: "client_secret_basic", client_secret: "..."}` | 机密客户端，通过 HTTP Basic 认证传递密钥 |
| `"client_secret_post"` | `{type: "client_secret_post", client_secret: "..."}` | 机密客户端，密钥在请求体中 |

如果您只有访问令牌且没有刷新能力，完全省略 `refresh`——它将在令牌过期前有效，然后代理失去访问权限。

> 💡 **获取 OAuth 令牌。** 如何获取初始访问令牌和刷新令牌取决于 MCP 服务器——请查阅其文档。一旦获得，使用上述结构将其存入保险库凭据；Anthropic 之后通过 `refresh.token_endpoint` 自动刷新。

**范围：** 保险库的范围是工作区。API 工作区中拥有开发者及以上角色的任何人都可以创建、读取（仅元数据——密钥是只写的）和附加保险库。`vault_ids` 可以在会话**创建**时设置，但不能通过会话更新设置（SDK 文档注释说"尚不支持；设置此字段的请求会被拒绝"）。

---

## 技能

技能是可复用的基于文件系统的资源，为您的代理提供领域专业知识：工作流、上下文和最佳实践，将通用代理转变为专家。与提示词（针对一次性任务的会话级指令）不同，技能按需加载，无需在多次会话中重复提供相同的指导。

两种类型——工作方式相同；代理在任务相关时自动使用它们：

| 类型 | 说明 |
|---|---|
| **Anthropic 预构建技能** | 常见文档任务（PowerPoint、Excel、Word、PDF）。按名称引用（如 `xlsx`）。 |
| **自定义技能** | 您通过技能 API 在组织中创建的技能。通过 `skill_id` + 可选 `version` 引用。 |

**每个代理最多 64 个技能。** 代理创建使用 `managed-agents-2026-04-01`；单独的技能 API（用于管理自定义技能定义）使用 `skills-2025-10-02`。

### 在会话中启用技能

技能通过 `agents.create()` 附加到**代理**定义：

```ts
const agent = await client.beta.agents.create(
  {
    name: "Financial Agent",
    model: "{{OPUS_ID}}",
    system: "You are a financial analysis agent.",
    skills: [
      { type: "anthropic", skill_id: "xlsx" },
      { type: "custom", skill_id: "skill_abc123", version: "latest" },
    ],
  }
);
```

Python：

```python
agent = client.beta.agents.create(
    name="Financial Agent",
    model="{{OPUS_ID}}",
    system="You are a financial analysis agent.",
    skills=[
        {"type": "anthropic", "skill_id": "xlsx"},
        {"type": "custom", "skill_id": "skill_abc123", "version": "latest"},
    ]
)
```

**技能引用字段：**

| 字段 | Anthropic 技能 | 自定义技能 |
|---|---|---|
| `type` | `"anthropic"` | `"custom"` |
| `skill_id` | 技能名称（如 `"xlsx"`、`"docx"`、`"pptx"`、`"pdf"`） | 来自技能 API 的技能 ID（如 `"skill_abc123"`） |
| `version` | — | `"latest"` 或特定版本号 |

### 技能 API

| 操作             | 方法   | 路径                                            |
| --------------------- | -------- | ----------------------------------------------- |
| 创建技能          | `POST`   | `/v1/skills`                                    |
| 列出技能           | `GET`    | `/v1/skills`                                    |
| 获取技能             | `GET`    | `/v1/skills/{id}`                               |
| 删除技能          | `DELETE` | `/v1/skills/{id}`                               |
| 创建版本        | `POST`   | `/v1/skills/{id}/versions`                      |
| 列出版本         | `GET`    | `/v1/skills/{id}/versions`                      |
| 获取版本           | `GET`    | `/v1/skills/{id}/versions/{version}`            |
| 删除版本         | `DELETE` | `/v1/skills/{id}/versions/{version}`            |
