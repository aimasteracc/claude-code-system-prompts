<!--
name: 'Data: Managed Agents endpoint reference'
description: Comprehensive reference for Managed Agents API endpoints, SDK methods, request/response schemas, error handling, and rate limits
ccVersion: 2.1.97
-->
# 托管代理 — 端点参考

所有端点都需要 `x-api-key` 和 `anthropic-version: 2023-06-01` 请求头。托管代理端点还需要 `anthropic-beta` 请求头。

## Beta 请求头

```
anthropic-beta: managed-agents-2026-04-01
```

SDK 会自动为所有 `client.beta.{agents,environments,sessions,vaults}.*` 调用添加此请求头。技能端点使用 `skills-2025-10-02`；文件端点使用 `files-api-2025-04-14`。

---

## SDK 方法参考

所有资源都在 `beta` 命名空间下。Python 和 TypeScript 共享相同的方法名。

| 资源 | Python / TypeScript（`client.beta.*`） | Go（`client.Beta.*`） |
| --- | --- | --- |
| 代理 | `agents.create` / `retrieve` / `update` / `list` / `archive` | `Agents.New` / `Get` / `Update` / `List` / `Archive` |
| 代理版本 | `agents.versions.list` | `Agents.Versions.List` |
| 环境 | `environments.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Environments.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 会话 | `sessions.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Sessions.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 会话事件 | `sessions.events.list` / `send` / `stream` | `Sessions.Events.List` / `Send` / `StreamEvents` |
| 会话资源 | `sessions.resources.add` / `retrieve` / `update` / `list` / `delete` | `Sessions.Resources.Add` / `Get` / `Update` / `List` / `Delete` |
| 保险库 | `vaults.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Vaults.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |
| 凭据 | `vaults.credentials.create` / `retrieve` / `update` / `list` / `delete` / `archive` | `Vaults.Credentials.New` / `Get` / `Update` / `List` / `Delete` / `Archive` |

**需要注意的命名差异：**
- 代理**没有删除**操作——只有 `archive`。其他资源两者都有。
- 会话资源使用 `add`（而非 `create`）。
- Go 的事件流是 `StreamEvents`（而非 `Stream`）。

**代理简写：** 会话创建时的 `agent` 字段接受纯字符串（`agent="agent_abc123"` — 使用最新版本）或完整引用对象（`{type: "agent", id: "agent_abc123", version: 123}`）。

**模型简写：** 代理创建时的 `model` 字段接受纯字符串（`model="claude-opus-4-6"` — 使用 `standard` 速度）或完整配置对象（`{type: "model_config", id: "claude-opus-4-6", speed: "fast"}`）。

---

## 代理

**每个流程的第一步。** 会话需要预先创建的代理——在 `managed-agents-2026-04-01` 下，会话中没有内联代理配置。

| 方法   | 路径                                             | 操作        | 说明                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/agents` | ListAgents | 列出代理 |
| `POST` | `/v1/agents` | CreateAgent | 创建保存的代理配置 |
| `GET` | `/v1/agents/{agent_id}` | GetAgent | 获取代理详情 |
| `POST` | `/v1/agents/{agent_id}` | UpdateAgent | 更新代理配置 |
| `POST` | `/v1/agents/{agent_id}/archive` | ArchiveAgent | 存档代理（代理没有硬删除） |
| `GET` | `/v1/agents/{agent_id}/versions` | ListAgentVersions | 列出代理版本 |

## 会话

| 方法   | 路径                                             | 操作        | 说明                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions` | ListSessions | 列出会话（分页） |
| `POST` | `/v1/sessions` | CreateSession | 创建新会话 |
| `GET` | `/v1/sessions/{session_id}` | GetSession | 获取会话详情 |
| `POST` | `/v1/sessions/{session_id}` | UpdateSession | 更新会话元数据/标题 |
| `DELETE` | `/v1/sessions/{session_id}` | DeleteSession | 删除会话 |
| `POST` | `/v1/sessions/{session_id}/archive` | ArchiveSession | 存档会话 |

## 事件

| 方法   | 路径                                             | 操作        | 说明                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/events` | ListEvents | 列出事件（轮询，分页） |
| `POST` | `/v1/sessions/{session_id}/events` | SendEvents | 发送事件（用户消息、工具结果） |
| `GET` | `/v1/sessions/{session_id}/events/stream` | StreamEvents | 通过 SSE 流式传输事件 |

## 会话资源

| 方法   | 路径                                                    | 操作        | 说明                              |
| -------- | ------------------------------------------------------- | ---------------- | ---------------------------------------- |
| `GET` | `/v1/sessions/{session_id}/resources` | ListResources | 列出附加到会话的资源 |
| `POST` | `/v1/sessions/{session_id}/resources` | AddResource | 附加文件或 github_repository 挂载（SDK 方法：`add`，而非 `create`） |
| `GET` | `/v1/sessions/{session_id}/resources/{resource_id}` | GetResource | 获取单个资源 |
| `POST` | `/v1/sessions/{session_id}/resources/{resource_id}` | UpdateResource | 更新资源 |
| `DELETE` | `/v1/sessions/{session_id}/resources/{resource_id}` | DeleteResource | 从会话中移除资源 |

## 环境

| 方法   | 路径                                                             | 操作            | 说明                         |
| -------- | ---------------------------------------------------------------- | -------------------- | ----------------------------------- |
| `POST`   | `/v1/environments`                                     | CreateEnvironment    | 创建环境                  |
| `GET`    | `/v1/environments`                                     | ListEnvironments     | 列出环境                   |
| `GET`    | `/v1/environments/{environment_id}`                    | GetEnvironment       | 获取环境详情             |
| `POST`   | `/v1/environments/{environment_id}`                    | UpdateEnvironment    | 更新环境                  |
| `DELETE` | `/v1/environments/{environment_id}`                    | DeleteEnvironment    | 删除环境，返回 204。 |
| `POST`   | `/v1/environments/{environment_id}/archive`            | ArchiveEnvironment   | 存档环境（只读；现有会话继续运行） |

## 保险库

保险库存储 Anthropic 代您管理的 MCP 凭据——支持自动刷新的 OAuth 凭据，或静态 Bearer 令牌。通过 `vault_ids` 附加到会话。概念指南和凭据格式详见 `managed-agents-tools.md` §保险库。

| 方法   | 路径                                             | 操作        | 说明                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/vaults`                                     | CreateVault      | 创建保险库                           |
| `GET`    | `/v1/vaults`                                     | ListVaults       | 列出保险库                              |
| `GET`    | `/v1/vaults/{vault_id}`                          | GetVault         | 获取保险库详情                        |
| `POST`   | `/v1/vaults/{vault_id}`                          | UpdateVault      | 更新保险库                             |
| `DELETE` | `/v1/vaults/{vault_id}`                          | DeleteVault      | 删除保险库                             |
| `POST`   | `/v1/vaults/{vault_id}/archive`                  | ArchiveVault     | 存档保险库                            |

## 凭据

凭据是存储在保险库中的单个密钥。

| 方法   | 路径                                                              | 操作          | 说明                  |
| -------- | ----------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/vaults/{vault_id}/credentials`                               | CreateCredential   | 创建凭据          |
| `GET`    | `/v1/vaults/{vault_id}/credentials`                               | ListCredentials    | 列出保险库中的凭据    |
| `GET`    | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | GetCredential      | 获取凭据元数据      |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | UpdateCredential   | 更新凭据            |
| `DELETE` | `/v1/vaults/{vault_id}/credentials/{credential_id}`               | DeleteCredential   | 删除凭据            |
| `POST`   | `/v1/vaults/{vault_id}/credentials/{credential_id}/archive`       | ArchiveCredential  | 存档凭据           |

## 文件

| 方法   | 路径                                             | 操作        | 说明                              |
| -------- | ------------------------------------------------ | ---------------- | ---------------------------------------- |
| `POST`   | `/v1/files`                            | UploadFile       | 上传文件                            |
| `GET`    | `/v1/files`                            | ListFiles        | 列出文件                               |
| `GET`    | `/v1/files/{file_id}`                  | GetFile          | 获取文件元数据（SDK 方法：`retrieve_metadata`） |
| `GET`    | `/v1/files/{file_id}/content`          | DownloadFile     | 下载文件内容                    |
| `DELETE` | `/v1/files/{file_id}`                  | DeleteFile       | 删除文件                            |

## 技能

| 方法   | 路径                                                            | 操作          | 说明                  |
| -------- | --------------------------------------------------------------- | ------------------ | ---------------------------- |
| `POST`   | `/v1/skills`                                          | CreateSkill        | 创建技能               |
| `GET`    | `/v1/skills`                                          | ListSkills         | 列出技能                  |
| `GET`    | `/v1/skills/{skill_id}`                               | GetSkill           | 获取技能详情            |
| `DELETE` | `/v1/skills/{skill_id}`                               | DeleteSkill        | 删除技能               |
| `POST`   | `/v1/skills/{skill_id}/versions`                      | CreateVersion      | 创建技能版本         |
| `GET`    | `/v1/skills/{skill_id}/versions`                      | ListVersions       | 列出技能版本          |
| `GET`    | `/v1/skills/{skill_id}/versions/{version}`            | GetVersion         | 获取技能版本            |
| `DELETE` | `/v1/skills/{skill_id}/versions/{version}`            | DeleteVersion      | 删除技能版本         |

---

## 请求/响应结构快速参考

### CreateAgent 请求体

**始终从此开始。** `model`、`system`、`tools`、`mcp_servers`、`skills` 是该对象的顶层字段——它们**不**放在会话上。

```json
{
  "name": "string (必填，1-256 字符)",
  "model": "{{OPUS_ID}} (必填 — 纯字符串，或 {id, speed} 对象)",
  "description": "string (可选，最多 2048 字符)",
  "system": "string (可选，最多 100,000 字符)",
  "tools": [
    { "type": "agent_toolset_20260401" }
  ],
  "skills": [
    { "type": "anthropic", "skill_id": "xlsx" },
    { "type": "custom", "skill_id": "skill_abc123", "version": "1" }
  ],
  "mcp_servers": [
    {
      "type": "url",
      "name": "github",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  ],
  "metadata": {
    "key": "value (最多 16 对，键 ≤64 字符，值 ≤512 字符)"
  }
}
```

> 限制：`tools` 最多 50 个，`skills` 最多 64 个，`mcp_servers` 最多 20 个（名称唯一）。

### CreateSession 请求体

```json
{
  "agent": "agent_abc123 (必填 — 最新版本的字符串简写，或 {type: \"agent\", id, version} 对象)",
  "environment_id": "env_abc123 (必填)",
  "title": "string (可选)",
  "resources": [
    {
      "type": "github_repository",
      "url": "https://github.com/owner/repo (必填)",
      "authorization_token": "ghp_... (必填)",
      "mount_path": "/workspace/repo (可选 — 默认为 /workspace/<仓库名>)",
      "checkout": { "type": "branch", "name": "main" }
    }
  ],
  "vault_ids": ["vlt_abc123 (可选 — 具有自动刷新的 MCP 凭据)"],
  "metadata": {
    "key": "value"
  }
}
```

> `agent` 字段只接受字符串 ID 或 `{type: "agent", id, version}` — `model`/`system`/`tools` 在代理上，而非此处。
>
> **`checkout`** 接受 `{type: "branch", name: "..."}` 或 `{type: "commit", sha: "..."}`。省略则使用仓库的默认分支。

### CreateEnvironment 请求体

```json
{
  "name": "string (必填)",
  "description": "string (可选)",
  "config": {
    "type": "cloud",
    "networking": {
      "type": "unrestricted | limited (联合类型 — 请查看 SDK 类型定义)"
    },
    "packages": { }
  },
  "metadata": { "key": "value" }
}
```

### SendEvents 请求体

```json
{
  "events": [
    {
      "type": "user.message",
      "content": [
        {
          "type": "text",
          "text": "Hello"
        }
      ]
    }
  ]
}
```

### 工具结果事件

```json
{
  "type": "user.custom_tool_result",
  "custom_tool_use_id": "sevt_abc123",
  "content": [{ "type": "text", "text": "Result data" }],
  "is_error": false
}
```

---

## 错误处理

托管代理端点使用标准 Anthropic API 错误格式。错误以 HTTP 状态码和包含 `type`、`error`、`request_id` 的 JSON 体返回：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Description of what went wrong"
  },
  "request_id": "req_011CRv1W3XQ8XpFikNYG7RnE"
}
```

向 Anthropic 报告问题时请提供 `request_id`——它能让我们端到端追踪请求。内部的 `error.type` 是以下值之一：

| 状态 | 错误类型 | 说明 |
|---|---|---|
| 400 | `invalid_request_error` | 请求格式错误或缺少必需参数 |
| 401 | `authentication_error` | API 密钥无效或缺失 |
| 403 | `permission_error` | API 密钥没有此操作的权限 |
| 404 | `not_found_error` | 请求的资源不存在 |
| 409 | `invalid_request_error` | 请求与资源的当前状态冲突（例如，向已存档的会话发送消息） |
| 413 | `request_too_large` | 请求体超过允许的最大大小 |
| 429 | `rate_limit_error` | 请求过多——检查速率限制请求头以获取重试时机 |
| 500 | `api_error` | 发生内部服务器错误 |
| 529 | `overloaded_error` | 服务暂时过载——使用退避策略重试 |

注意，`409 Conflict` 携带 `error.type: "invalid_request_error"`（没有单独的 `conflict_error` 类型）；同时检查 HTTP 状态和 `message` 以区分冲突和其他无效请求。

---

## 速率限制

托管代理端点有按组织计算的每分钟请求（RPM）限制，与您的[消息 API 词元限制](https://platform.claude.com/docs/en/api/rate-limits)是分开的。会话内的模型推理仍从您组织的标准 ITPM/OTPM 限制中扣除。

| 端点组 | 范围 | RPM | 最大并发数 |
|---|---|---|---|
| 创建操作（代理、会话、保险库） | 组织 | 60 | — |
| 所有其他操作（代理、会话、保险库） | 组织 | 600 | — |
| 所有操作（环境） | 组织 | 60 | 5 |

文件和技能端点使用标准基于层级的[速率限制](https://platform.claude.com/docs/en/api/rate-limits)。

当超出限制时，API 返回 `429` 并附带 `rate_limit_error`（响应信封详见[错误处理](#错误处理)）以及 `retry-after` 请求头，指示重试前需等待的秒数。Anthropic SDK 会读取此请求头并自动重试。
