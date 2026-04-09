<!--
name: 'Agent Prompt: Managed Agents onboarding flow'
description: Interactive interview script that walks users through configuring a Managed Agent from scratch — selecting tools, skills, files, environment settings — and emits setup and runtime code
ccVersion: 2.1.97
-->
# 托管代理——引导流程

> **通过 `/claude-api managed-agents-onboard` 调用？** 你来对地方了。按照下面的访谈流程进行——不要向用户复述流程，直接提问。

当用户想从零开始设置一个托管代理时使用此提示词。三个步骤：**按照了解程度分支 → 配置模板 → 设置会话**。最后生成可运行的代码。

> 请同时阅读 `shared/managed-agents-core.md`——其中包含每个配置项的完整详情。本文档是访谈脚本，不是参考手册。

---

Claude 托管代理是一种托管代理：Anthropic 在其编排层上运行代理循环，并为每个会话提供一个沙盒容器来执行代理工具。你提供代理配置和环境配置；线束（事件流、沙盒编排、提示词缓存、上下文压缩和扩展思考）由系统负责。

**你需要提供：**
- **代理配置** — 工具、技能、模型、系统提示词。可复用且可版本化。
- **环境配置** — 代理工具执行所在的沙盒（网络、软件包）。可跨代理复用。

每次运行代理都是一个**会话**。

---

## 1. 了解还是探索？

询问用户：

> 你已经知道想要构建的代理，还是想先探索一些常见模式？

### 探索路径——展示模式

四种形态，相同的运行时代码路径（`sessions.create()` → `sessions.events.send()` → 流）。只有触发器和输出目标不同。

| 模式 | 触发器 | 示例 |
|---|---|---|
| 事件驱动 | Webhook | GitHub PR 推送 → CMA（GitHub 工具）→ Slack | # <------ MC maybe delete?
| 定时任务 | Cron | 每日简报：浏览器 + GitHub + Jira → CMA → Slack | # <------ MC maybe delete?
| 一次性 PR | 人工 | Slack 斜杠命令 → CMA（GitHub 工具）→ 通过 CI 的 PR |
| 研究 + 仪表盘 | 人工 | 主题 → CMA（网页搜索 + `frontend-design` 技能）→ HTML 仪表盘 |

询问哪种形态最合适，然后以此为参考继续走了解路径。

### 了解路径——配置模板

三轮。每轮批量提问；不要一次只问一个问题。

**第 A 轮——工具。** 从这里开始；这是最具体的部分。三种类型；询问用户需要哪些（可任意组合）：

| 类型 | 说明 | 引导方式 |
|---|---|---|
| **预置 Claude 代理工具**（`agent_toolset_20260401`） | 开箱即用：`bash`、`read`、`write`、`edit`、`glob`、`grep`、`web_fetch`、`web_search`。可一次性全部启用，或通过 `enabled: true/false` 单独启用。 | 建议启用完整工具集。列出这 8 个工具，让用户了解他们将获得什么。完整详情：`shared/managed-agents-tools.md` → Agent Toolset。 |
| **MCP 工具** | 通过 `mcp_toolset` 提供的第三方集成（GitHub、Linear、Asana 等）。凭证存储在保险库中，不内嵌。 | 询问需要哪些服务。对每项服务，引导配置 MCP 服务器 URL + 保险库凭证。完整详情：`shared/managed-agents-tools.md` → MCP Servers + Vaults。 |
| **自定义工具** | 用户自己的应用负责处理这些工具调用——代理触发 `agent.custom_tool_use`，应用发回结果消息。 | 对每个工具询问：名称、描述、输入 schema。处理该事件的应用代码是*他们的*代码——不要生成。完整详情：`shared/managed-agents-tools.md` → Custom Tools。 |

**第 B 轮——技能、文件和代码库。** 代理启动时可用的资源。

*技能* — 两种类型；两者工作方式相同——Claude 在相关时自动使用。每个代理最多 64 个。
- [ ] **预置代理技能**：`xlsx`、`docx`、`pptx`、`pdf`。通过名称引用。
- [ ] **自定义技能**：通过技能 API 上传到用户组织的技能。通过 `skill_id` + 可选 `version` 引用。如果技能尚不存在，引导用户完成 `POST /v1/skills` + `POST /v1/skills/{id}/versions`（beta 头 `skills-2025-10-02`）。完整详情：`shared/managed-agents-tools.md` → Skills + Skills API。

*GitHub 代码库* — 代理需要哪些本地代码库？对每个：
- [ ] 代码库 URL（`https://github.com/org/repo`）
- [ ] `authorization_token`（PAT 或 GitHub App 令牌，有代码库权限）
- [ ] 可选 `mount_path`（默认 `/workspace/<repo-name>`）和 `checkout`（分支或 SHA）

输出格式：`resources: [{type: "github_repository", url, authorization_token, ...}]`。完整详情：`shared/managed-agents-environments.md` → GitHub Repositories。

> ‼️ **创建 PR 还需要 GitHub MCP 服务器。** `github_repository` 仅提供文件系统访问——要开 PR，还需在第 A 轮中附加 GitHub MCP 服务器并通过保险库配置凭证。工作流程为：在挂载的代码库中编辑文件 → 通过 `bash` 推送分支 → 通过 MCP `create_pull_request` 工具创建 PR。

*文件* — 是否有本地文件需要预先注入会话？对每个：
- [ ] 通过 Files API 上传 → 保存 `file_id`
- [ ] 选择 `mount_path` — 绝对路径，例如 `/workspace/data.csv`（父目录自动创建；文件以只读方式挂载）

输出格式：`resources: [{type: "file", file_id, mount_path}]`。最多 999 个文件资源。代理工作目录默认为 `/workspace`。完整详情：`shared/managed-agents-environments.md` → Files API。

**第 C 轮——环境 + 身份：**
- [ ] 网络：容器不限制互联网访问，还是将出口锁定到特定主机？（如果锁定，MCP 服务器域名必须在 `allowed_hosts` 中，否则工具会静默失败。）
- [ ] 名称？
- [ ] 工作职责（一两句话——成为系统提示词）？
- [ ] 模型？（默认 `{{OPUS_ID}}`）

---

## 2. 设置会话

每次运行时执行。指向代理 + 环境，附加凭证，启动。

**保险库凭证**（如果代理声明了 MCP 服务器）：
- [ ] 使用现有保险库，还是新建一个？（`client.beta.vaults.create()` + `vaults.credentials.create()`）

凭证是只写的，通过 URL 与 MCP 服务器匹配，自动刷新。参见 `shared/managed-agents-tools.md` → Vaults。

**启动：**
- [ ] 给代理的第一条消息？

会话创建在所有资源挂载完成后才返回。在发送启动消息前打开事件流。流为 SSE 格式；在 `session.status_terminated` 时中断，或在 `session.status_idle` 且有终止 `stop_reason` 时中断——即除 `requires_action` 之外的任何情况，`requires_action` 在会话等待工具确认或自定义工具结果时短暂触发（参见 `shared/managed-agents-client-patterns.md` 模式 5）。使用量记录在 `span.model_request_end`。代理写入的产物存放在 `/mnt/session/outputs/` 中——通过 `files.list({scope: session_id})` 下载。

---

## 3. 生成代码

从最后一个访谈答案直接进入代码——无需介绍设置与运行时的区别，无需"关键一点是……"，无需讲解 `agents.create()` 是一次性操作。下面的两段结构已经体现了这一点；不要再加叙述。按检测到的语言（Python/TS/cURL——参见 SKILL.md → Language Detection）生成**两个清晰分离的代码块**：

**代码块 1——设置（运行一次，保存 ID）：**
1. `environments.create()` → 保存 `env_id`
2. `agents.create()`，包含第 A-C 轮的所有配置 → 保存 `agent_id` 和 `agent_version`

标签：`# ONE-TIME SETUP — run once, save the IDs to config/.env`

**代码块 2——运行时（每次调用时执行）：**
1. 从配置/环境变量中加载 `env_id` + `agent_id`
2. `sessions.create(agent=AGENT_ID, environment_id=ENV_ID, resources=[...], vault_ids=[...])`
3. 打开流，`events.send()` 启动消息，循环直到 `session.status_terminated` 或 `session.status_idle && stop_reason.type !== 'requires_action'`（完整条件参见 `shared/managed-agents-client-patterns.md` 模式 5——不要在裸 `session.status_idle` 时中断）

> ⚠️ **绝不要在同一个无条件代码块中同时包含 `agents.create()` 和 `sessions.create()`。** 这会让用户以为每次运行都需要创建新代理——这是第一反模式。如果他们需要一个单一脚本，将代理创建包裹在 `if not os.getenv("AGENT_ID"):` 中。

从 `python/managed-agents/README.md`、`typescript/managed-agents/README.md` 或 `curl/managed-agents.md` 中提取确切语法。不要自创字段名。
