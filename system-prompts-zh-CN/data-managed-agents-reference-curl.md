<!--
name: 'Data: Managed Agents reference — cURL'
description: Provides cURL and raw HTTP request examples for the Managed Agents API including environment, agent, and session lifecycle operations
ccVersion: 2.1.97
-->
# 托管代理 — cURL / 原始 HTTP

当用户需要原始 HTTP 请求或在没有 SDK 的情况下工作时，使用以下示例。

## 初始化

```bash
export ANTHROPIC_API_KEY="your-api-key"

# 公共请求头
HEADERS=(
  -H "Content-Type: application/json"
  -H "x-api-key: $ANTHROPIC_API_KEY"
  -H "anthropic-version: 2023-06-01"
  -H "anthropic-beta: managed-agents-2026-04-01"
)
```

---

## 创建环境

```bash
curl -X POST https://api.anthropic.com/v1/environments \
  "${HEADERS[@]}" \
  -d '{
    "name": "my-dev-env",
    "config": {
      "type": "cloud",
      "networking": { "type": "unrestricted" }
    }
  }'
```

### 使用受限网络

```bash
curl -X POST https://api.anthropic.com/v1/environments \
  "${HEADERS[@]}" \
  -d '{
    "name": "restricted-env",
    "config": {
      "type": "cloud",
      "networking": {
        "type": "package_managers_and_custom",
        "allowed_hosts": ["api.example.com"]
      }
    }
  }'
```

---

## 创建代理（必须的第一步）

> ⚠️ **没有内联代理配置。** 在 `managed-agents-2026-04-01` 下，`model`/`system`/`tools` 是 `POST /v1/agents` 的顶层字段，而非会话上的字段。始终先创建代理——会话只接受 `"agent": {"type": "agent", "id": "..."}`。

### 最小化配置

```bash
# 1. 创建代理
curl -X POST https://api.anthropic.com/v1/agents \
  "${HEADERS[@]}" \
  -d '{
    "name": "Coding Assistant",
    "model": "{{OPUS_ID}}",
    "tools": [{ "type": "agent_toolset_20260401" }]
  }'
# → { "id": "agent_abc123", ... }

# 2. 启动会话
curl -X POST https://api.anthropic.com/v1/sessions \
  "${HEADERS[@]}" \
  -d '{
    "agent": { "type": "agent", "id": "agent_abc123", "version": "1772585501101368014" },
    "environment_id": "env_abc123"
  }'
```

### 带系统提示词、自定义工具和 GitHub 仓库

```bash
# 1. 创建代理
curl -X POST https://api.anthropic.com/v1/agents \
  "${HEADERS[@]}" \
  -d '{
    "name": "Code Reviewer",
    "model": "{{OPUS_ID}}",
    "system": "You are a senior code reviewer. Be thorough and constructive.",
    "tools": [
      { "type": "agent_toolset_20260401" },
      {
        "type": "custom",
        "name": "run_linter",
        "description": "Run the project linter on a file",
        "input_schema": {
          "type": "object",
          "properties": {
            "file_path": { "type": "string", "description": "Path to lint" }
          },
          "required": ["file_path"]
        }
      }
    ]
  }'

# 2. 启动挂载了仓库的会话
curl -X POST https://api.anthropic.com/v1/sessions \
  "${HEADERS[@]}" \
  -d '{
    "agent": { "type": "agent", "id": "agent_abc123", "version": "1772585501101368014" },
    "environment_id": "env_abc123",
    "title": "Code review session",
    "resources": [
      {
        "type": "github_repository",
        "url": "https://github.com/owner/repo",
        "mount_path": "/workspace/repo",
        "authorization_token": "ghp_...",
        "branch": "feature-branch"
      }
    ]
  }'
```

---

## 发送用户消息

```bash
curl -X POST https://api.anthropic.com/v1/sessions/$SESSION_ID/events \
  "${HEADERS[@]}" \
  -d '{
    "events": [
      {
        "type": "user.message",
        "content": [{ "type": "text", "text": "Review the auth module for security issues" }]
      }
    ]
  }'
```

---

## 流式传输事件（SSE）

```bash
curl -N https://api.anthropic.com/v1/sessions/$SESSION_ID/events/stream \
  "${HEADERS[@]}"
```

响应格式：

```
event: session.status_running
data: {"type":"session.status_running","id":"sevt_...","processed_at":"..."}

event: agent.message
data: {"type":"agent.message","id":"sevt_...","content":[{"type":"text","text":"I'll review..."}],"processed_at":"..."}

event: session.status_idle
data: {"type":"session.status_idle","id":"sevt_...","processed_at":"..."}
```

---

## 轮询事件

```bash
# 获取所有事件
curl https://api.anthropic.com/v1/sessions/$SESSION_ID/events \
  "${HEADERS[@]}"

# 分页——获取下一页事件
curl "https://api.anthropic.com/v1/sessions/$SESSION_ID/events?page=page_abc123" \
  "${HEADERS[@]}"
```

---

## 提供自定义工具结果

当代理调用自定义工具时，发回结果：

```bash
curl -X POST https://api.anthropic.com/v1/sessions/$SESSION_ID/events \
  "${HEADERS[@]}" \
  -d '{
    "events": [
      {
        "type": "user.custom_tool_result",
        "custom_tool_use_id": "sevt_abc123",
        "content": [{ "type": "text", "text": "No linting errors found." }]
      }
    ]
  }'
```

---

## 中断运行中的会话

```bash
curl -X POST https://api.anthropic.com/v1/sessions/$SESSION_ID/events \
  "${HEADERS[@]}" \
  -d '{
    "events": [
      {
        "type": "interrupt"
      }
    ]
  }'
```

---

## 获取会话详情

```bash
curl https://api.anthropic.com/v1/sessions/$SESSION_ID \
  "${HEADERS[@]}"
```

---

## 列出会话

```bash
curl https://api.anthropic.com/v1/sessions \
  "${HEADERS[@]}"
```

---

## 删除会话

```bash
curl -X DELETE https://api.anthropic.com/v1/sessions/$SESSION_ID \
  "${HEADERS[@]}"
```

---

## 上传文件

```bash
curl -X POST https://api.anthropic.com/v1/files \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: files-api-2025-04-14" \
  -F "file=@path/to/file.txt" \
  -F "purpose=agent"
```

---

## 列出和下载会话文件

列出代理在会话期间写入 `/mnt/session/outputs/` 的文件，然后下载它们。

```bash
# 列出与会话关联的文件
curl "https://api.anthropic.com/v1/files?scope=$SESSION_ID" \
  "${HEADERS[@]}"

# 下载特定文件
curl "https://api.anthropic.com/v1/files/$FILE_ID/content" \
  "${HEADERS[@]}" \
  -o downloaded_file.txt
```

---

## 列出代理

```bash
curl https://api.anthropic.com/v1/agents \
  "${HEADERS[@]}"
```

---

## MCP 服务器集成

```bash
# 1. 代理声明 MCP 服务器（此处无认证——认证放在保险库中）
curl -X POST https://api.anthropic.com/v1/agents \
  "${HEADERS[@]}" \
  -d '{
    "name": "MCP Agent",
    "model": "{{OPUS_ID}}",
    "mcp_servers": [
      { "type": "url", "name": "my-tools", "url": "https://my-mcp-server.example.com/sse" }
    ],
    "tools": [
      { "type": "agent_toolset_20260401" },
      { "type": "mcp_toolset", "mcp_server_name": "my-tools" }
    ]
  }'

# 2. 会话附加包含该 MCP 服务器 URL 凭据的保险库
curl -X POST https://api.anthropic.com/v1/sessions \
  "${HEADERS[@]}" \
  -d '{
    "agent": "agent_abc123",
    "environment_id": "env_abc123",
    "vault_ids": ["vlt_abc123"]
  }'
```

创建保险库和添加凭据的方法详见 `shared/managed-agents-tools.md` §保险库。

---

## 工具配置

```bash
curl -X POST https://api.anthropic.com/v1/agents \
  "${HEADERS[@]}" \
  -d '{
    "name": "Restricted Agent",
    "model": "{{OPUS_ID}}",
    "tools": [
      {
        "type": "agent_toolset_20260401",
        "default_config": { "enabled": true },
        "configs": [
          { "name": "bash", "enabled": false }
        ]
      }
    ]
  }'
```
