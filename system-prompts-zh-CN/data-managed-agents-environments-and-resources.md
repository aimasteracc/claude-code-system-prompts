<!--
name: 'Data: Managed Agents environments and resources'
description: Reference documentation covering Managed Agents environments, file resources, GitHub repository mounting, and the Files API with SDK examples
ccVersion: 2.1.97
-->
# 托管代理 — 环境与资源

## 环境

创建会话需要 `environment_id`。环境是在 Anthropic 基础设施中启动容器的**可复用配置模板**——您可以为不同用例创建不同的环境（例如数据可视化与 Web 开发，使用不同的软件包集）。Anthropic 负责处理扩缩容、容器生命周期和工作编排。

**环境名称必须唯一。** 使用已存在的名称创建环境会返回 409。

### 网络配置

| 网络策略                  | 说明                                                   |
| ------------------------------- | ------------------------------------------------------------- |
| `unrestricted`                  | 完全出站访问（法律黑名单除外）                          |
| `package_managers_and_custom`   | 包管理器 + 自定义 `allowed_hosts`                      |

```json
{
  "networking": {
    "type": "package_managers_and_custom",
    "allowed_hosts": ["api.example.com"]
  }
}
```

**MCP 注意事项：** 如果使用受限网络，请确保 `allowed_hosts` 包含您的 MCP 服务器域名。否则容器无法访问它们，工具会静默失败。

### 创建环境

SDK 会自动添加 `managed-agents-2026-04-01`。TypeScript 示例：

```ts
const env = await client.beta.environments.create({
  name: "my_env",
  config: {
    type: "cloud",
    networking: { type: "unrestricted" },
  },
});
```

### 环境 CRUD

| 操作        | 方法   | 路径                                       | 说明 |
| ---------------- | -------- | ------------------------------------------ | ----- |
| 创建           | `POST`   | `/v1/environments`                         | |
| 列表             | `GET`    | `/v1/environments`                         | 分页（`limit`、`after_id`、`before_id`） |
| 获取              | `GET`    | `/v1/environments/{id}`                    | |
| 更新           | `POST`   | `/v1/environments/{id}`                    | 变更仅对**新**容器生效；现有会话保留其原始配置 |
| 删除           | `DELETE` | `/v1/environments/{id}`                    | 返回 204。 |
| 存档          | `POST`   | `/v1/environments/{id}/archive`            | 只读。无法创建新会话；现有会话继续运行。 |

---

## 资源

向会话附加文件和 GitHub 仓库。**会话创建会阻塞直到所有资源挂载完成**——容器在每个文件和仓库就位之前不会变为 `running` 状态。每个会话最多 **999 个文件资源**。每个会话支持多个 GitHub 仓库。

### 文件上传（输入 — 主机 → 代理）

首先通过文件 API 上传文件，然后通过 `file_id` + `mount_path` 引用：

```ts
// 1. 上传
const file = await client.beta.files.upload({
  file: fs.createReadStream("data.csv"),
  purpose: "agent",
});

// 2. 作为会话资源附加
const session = await client.beta.sessions.create({
  agent: agent.id,
  environment_id: envId,
  resources: [
    { type: "file", file_id: file.id, mount_path: "/workspace/data.csv" }
  ],
});
```

**`mount_path` 是必填项**，必须是绝对路径。父目录会自动创建。代理工作目录默认为 `/workspace`。文件以只读方式挂载——代理将修改版本写入新路径。

### 会话输出（输出 — 代理 → 主机）

代理可以在会话期间将文件写入 `/mnt/session/outputs/`。这些文件会被文件 API 自动捕获，之后可以列出和下载：

```ts
// 轮次完成后，列出该会话范围内的输出文件：
for await (const f of client.beta.files.list({ scope: session.id })) {
  console.log(f.filename, f.size_bytes);
  const resp = await client.beta.files.download(f.id);
  const text = await resp.text();
}
```

**要求：**
- 必须为代理启用 `write` 工具（或 `bash`）才能创建输出文件。
- 会话范围的 `files.list` / `files.download` 捕获写入 `/mnt/session/outputs/` 的输出。
- `session_id` 是 `files.list` 上的查询过滤器（SDK 类型中尚未支持——通过类型转换或展开使用）。
- 从 `session.status_idle` 到输出文件出现在 `files.list` 中之间有短暂的索引延迟（约 1-3 秒）。如果列表为空，重试一两次。

这为您提供了双向文件桥接：上传参考数据，下载代理产物。

### GitHub 仓库

在初始化期间、代理开始执行之前，将 GitHub 仓库克隆到会话容器中。代理可以通过 `bash`（`git`）读取、编辑、提交和推送。每个会话支持多个仓库——每个仓库添加一个 `resources` 条目。

**字段：**

| 字段 | 必填 | 说明 |
|---|---|---|
| `type` | ✅ | `"github_repository"` |
| `url` | ✅ | GitHub 仓库 URL |
| `authorization_token` | ✅ | 具有仓库访问权限的 GitHub 个人访问令牌。**API 响应中永不回显。** |
| `mount_path` | ❌ | 仓库将被克隆到的路径。默认为 `/workspace/<仓库名>`。 |
| `checkout` | ❌ | `{type: "branch", name: "..."}` 或 `{type: "commit", sha: "..."}`。默认为仓库的默认分支。 |

**令牌权限级别**（精细化 PAT）：
- `Contents: Read` — 仅克隆
- `Contents: Read and write` — 推送变更和创建拉取请求

> ‼️ **要生成拉取请求**，还需要 GitHub **MCP 服务器**访问——`github_repository` 资源只提供文件系统访问。详见 `shared/managed-agents-tools.md` → MCP 服务器。PR 工作流是：在挂载的仓库中编辑文件 → 通过 `bash` 推送分支 → 通过 MCP `create_pull_request` 工具创建 PR。

**TypeScript：**

```ts
// 1. 创建代理 — 声明 GitHub MCP（此处无认证）
const agent = await client.beta.agents.create(
  {
    name: 'GitHub Agent',
    model: '{{OPUS_ID}}',
    mcp_servers: [
      { type: 'url', name: 'github', url: 'https://api.githubcopilot.com/mcp/' },
    ],
    tools: [
      { type: 'agent_toolset_20260401', default_config: { enabled: true } },
      { type: 'mcp_toolset', mcp_server_name: 'github' },
    ],
  },
);

// 2. 启动会话 — 附加保险库用于 MCP 认证 + 挂载仓库
const session = await client.beta.sessions.create({
  agent: agent.id,
  environment_id: envId,
  vault_ids: [vaultId],  // 保险库包含 GitHub MCP OAuth 凭据
  resources: [
    {
      type: 'github_repository',
      url: 'https://github.com/owner/repo',
      authorization_token: process.env.GITHUB_TOKEN,  // 仓库克隆令牌（≠ MCP 认证）
      checkout: { type: 'branch', name: 'main' },
    },
  ],
});
```

**Python：**

```python
import os

agent = client.beta.agents.create(
    name="GitHub Agent",
    model="{{OPUS_ID}}",
    mcp_servers=[{
        "type": "url",
        "name": "github",
        "url": "https://api.githubcopilot.com/mcp/",
    }],
    tools=[
        {"type": "agent_toolset_20260401", "default_config": {"enabled": True}},
        {"type": "mcp_toolset", "mcp_server_name": "github"},
    ],
)

session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=env_id,
    vault_ids=[vault_id],  # 保险库包含 GitHub MCP OAuth 凭据
    resources=[{
        "type": "github_repository",
        "url": "https://github.com/owner/repo",
        "authorization_token": os.environ["GITHUB_TOKEN"],  # 仓库克隆令牌（≠ MCP 认证）
        "checkout": {"type": "branch", "name": "main"},
    }],
)
```

---

## 文件 API

上传和管理用于会话资源的文件，以及下载代理写入 `/mnt/session/outputs/` 的文件。

| 操作        | 方法   | 路径                                  | SDK |
| ---------------- | -------- | ------------------------------------- | --- |
| 上传           | `POST`   | `/v1/files`                           | `client.beta.files.upload({ file })` |
| 列表             | `GET`    | `/v1/files?session_id=...`            | `client.beta.files.list({ session_id })` |
| 获取元数据     | `GET`    | `/v1/files/{id}`                      | `client.beta.files.retrieveMetadata(id)` |
| 下载         | `GET`    | `/v1/files/{id}/content`              | `client.beta.files.download(id)` → `Response` |
| 删除           | `DELETE` | `/v1/files/{id}`                      | `client.beta.files.delete(id)` |

列表上的 `session_id` 过滤器将结果限定在该会话写入 `/mnt/session/outputs/` 的文件。不使用过滤器时，您将获得上传到您账户的所有文件。
