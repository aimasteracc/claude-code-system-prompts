<!--
name: 'Data: Managed Agents core concepts'
description: Reference documentation for the Managed Agents API covering core concepts (Agents, Sessions, Environments, Containers), lifecycle, versioning, endpoints, and usage patterns
ccVersion: 2.1.97
-->
# 托管代理 — 核心概念

## 架构

托管代理围绕四个核心概念构建：

| 概念 | 端点 | 说明 |
|---|---|---|
| **代理** | `/v1/agents` | 定义代理能力和角色的持久化版本化对象：模型、系统提示词、工具、MCP 服务器、技能。**必须在启动会话之前创建。** 详见下方代理部分。 |
| **会话** | `/v1/sessions` | 与代理进行的有状态交互。通过 ID 引用预先创建的代理 + 环境 + 初始指令，并生成事件流。 |
| **环境** | `/v1/environments` | 定义容器配置的模板。 |
| **容器** | 不适用 | 代理**工具**执行的隔离计算实例（bash、文件操作、代码）。代理循环不在此运行——它运行在 Anthropic 的编排层，并通过工具调用对容器执行操作。 |

```
                       ┌─────────────────────────────────────┐
                       │  Anthropic 编排层                   │
代理（配置） ─────────▶│  （代理循环：Claude + 工具调用）      │
                       └──────────────┬──────────────────────┘
                                      │ 工具调用
                                      ▼
环境（模板） ──▶ 容器（工具执行工作空间）
                                 │
                         会话 ───┤
                                 ├── 资源（文件、仓库——启动时挂载）
                                 ├── 保险库 ID（MCP 凭据引用）
                                 └── 会话（事件流双向输入/输出）
```

> **创建代理是前提条件。** 会话通过 ID 引用预先创建的代理——`model`/`system`/`tools` 属于代理对象，绝不放在会话上。每个流程都从 `POST /v1/agents` 开始。

---

## 会话生命周期

```
rescheduling → running ↔ idle → terminated
```

| 状态         | 说明                                                        |
| -------------- | ------------------------------------------------------------------ |
| `idle` | 代理已完成当前任务，正在等待输入。它要么等待通过 `user.message` 继续工作的输入，要么等待 `user.custom_tool_result` 或 `user.tool_confirmation`。附带的 `stop_reason` 包含代理停止工作原因的更多信息。 |
| `running` | 会话已开始运行，代理正在积极工作。 |
| `rescheduling` | 在发生可重试错误后，会话正在（重新）调度，等待编排系统接管。 |
| `terminated` | 会话已终止，进入不可逆且不可用的状态。  |

- 事件可在会话处于 `running` 或 `idle` 状态时发送。消息排队后按顺序处理。
- 代理在收到新事件时从 `idle` 转为 `running`，完成后再次转为 `idle`。
- 错误以 `session.error` 事件的形式出现在流中，而非作为状态值。

### 会话内置功能

- **上下文压缩** — 如果接近最大上下文窗口，API 会自动压缩会话历史以保持交互持续进行
- **提示词缓存** — 历史重复词元会被缓存，减少处理时间和费用
- **深度思考** — 默认开启，以 `agent.thinking` 事件的形式返回

### 会话操作

| 操作 | 说明 |
|---|---|
| 列表 / 获取 | 分页列表或按 ID 获取单个资源 |
| 更新 | 只有 `title` 可更新 |
| 存档 | 会话变为**只读**，不可逆。 |
| 删除 | 永久删除会话、事件历史、容器和检查点。 |

---

## 会话

会话是在环境内运行的代理实例。

### 会话对象

API 返回的关键字段：

| 字段           | 类型     | 说明                                         |
| --------------- | -------- | --------------------------------------------------- |
| `type` | string | 始终为 `"session"` |
| `id` | string | 唯一会话 ID |
| `title` | string | 人类可读的标题 |
| `status` | string | `idle`、`running`、`rescheduling`、`terminated` |
| `created_at` | string | ISO 8601 时间戳 |
| `updated_at` | string | ISO 8601 时间戳 |
| `archived_at` | string | ISO 8601 时间戳（可为空） |
| `environment_id` | string | 环境 ID |
| `agent` | object | 代理配置 |
| `resources` | array | 附加的文件和仓库 |
| `metadata` | object | 用户提供的键值对（最多 8 对） |
| `usage` | object | 词元使用统计 |

### 创建会话

**没有代理的会话毫无意义。** 会话通过 ID 引用预先创建的代理。首先通过 `agents.create()` 创建代理，然后引用它：

```ts
// 1. 创建代理（可复用、版本化）
const agent = await client.beta.agents.create(
  {
    name: "Coding Assistant",
    model: "{{OPUS_ID}}",
    system: "You are a helpful coding agent.",
    tools: [{ type: "agent_toolset_20260401"}],
  },
);

// 2. 启动引用该代理的会话
const session = await client.beta.sessions.create(
  {
    agent: agent.id,  // 字符串简写 → 最新版本。或：{ type: "agent", id: agent.id, version: agent.version }
    environment_id: environmentId,
    title: "Hello World Session",
  },
);
```

**会话创建参数：**

| 字段           | 类型     | 必填 | 说明                                    |
| --------------- | -------- | -------- | ---------------------------------------------- |
| `agent`         | string 或 object | **是** | 字符串简写 `"agent_abc123"`（最新版本）或 `{type: "agent", id, version}` |
| `environment_id`| string   | **是**  | 环境 ID                                 |
| `title`         | string   | 否       | 人类可读的名称（出现在日志/仪表盘中） |
| `resources`     | array    | 否       | 文件或 GitHub 仓库，在启动时挂载到容器 |
| `vault_ids`     | array    | 否       | 保险库 ID（`vlt_*`）— 具有自动刷新的 MCP 凭据。详见 `shared/managed-agents-tools.md` → 保险库。 |
| `metadata`      | object   | 否       | 用户提供的键值对                  |

**代理配置字段**（传给 `agents.create()`，而非 `sessions.create()`）：

| 字段         | 类型     | 必填 | 说明                                    |
| ------------- | -------- | -------- | ---------------------------------------------- |
| `name`        | string   | **是**  | 人类可读的名称（1-256 个字符）              |
| `model`       | string 或 object | **是** | Claude 模型 ID（纯字符串，或 `{id, speed}` 对象）。支持所有 Claude 4.5+ 模型。 |
| `system`      | string   | 否   | 系统提示词——定义代理的行为（最多 100K 字符） |
| `tools`       | array    | 否   | 包含三种类型：(1) 预构建 Claude 代理工具（`agent_toolset_20260401`），(2) MCP 工具（`mcp_toolset`），(3) 自定义客户端工具。最多 128 个。 |
| `mcp_servers` | array    | 否   | MCP 服务器连接——标准化的第三方能力（如 GitHub、Asana）。最多 20 个，名称唯一。详见 `shared/managed-agents-tools.md` → MCP 服务器。 |
| `skills`      | array    | 否   | 具有渐进式披露的自定义"最佳实践"上下文。最多 64 个。详见 `shared/managed-agents-tools.md` → 技能。 |
| `description` | string   | 否   | 代理的描述（最多 2048 字符）    |
| `metadata`    | object   | 否   | 任意键值对（最多 16 对，键 ≤64 字符，值 ≤512 字符） |

---

## 代理

**这是每个托管代理流程的起点。** 代理对象是一个持久化的版本化配置——您只需创建一次，然后每次启动会话时通过 ID 引用它。没有代理就没有会话。

### 代理对象

API 是**扁平的** — `model`、`system`、`tools` 等是顶层字段，不包裹在 `agent:{}` 子对象中。

| 字段              | 类型     | 必填 | 说明                                        |
| ------------------ | -------- | -------- | -------------------------------------------------- |
| `name`             | string   | 是      | 人类可读的名称                                |
| `model`            | string   | 是      | Claude 模型 ID                                    |
| `system`           | string   | 否      | 系统提示词                                      |
| `tools`            | array    | 否      | 代理工具集 / MCP 工具集 / 自定义工具         |
| `mcp_servers`      | array    | 否      | MCP 服务器连接                             |
| `skills`           | array    | 否      | 技能引用（最多 64 个）                          |
| `description`      | string   | 否      | 代理的描述                           |
| `metadata`         | object   | 否      | 任意键值对                          |

### 生命周期：创建一次，运行多次，就地更新

代理是**持久化资源**，而非每次运行的参数。预期模式：

```
┌─ 初始化（一次） ───────┐     ┌─ 运行时（每次调用） ──────────┐
│ agents.create()        │     │ sessions.create(             │
│   → 保存 agent_id      │ ──→ │   agent={type:..., id: ID}   │
│     到配置/环境/数据库  │     │ )                            │
└────────────────────────┘     └──────────────────────────────┘
```

**反模式：** 在每次脚本运行的顶部调用 `agents.create()`。这会积累孤立的代理对象，在每次调用时承担创建延迟，并破坏版本管理模型。如果您看到 `agents.create()` 出现在每次请求或每次定时任务调用的函数中，那是错误的——应将其提升为一次性初始化并持久化 ID。

### 版本管理

每次 `POST /v1/agents/{id}`（更新）都会创建一个新的不可变版本（数字时间戳，例如 `1772585501101368014`）。代理的历史是只追加的——无法编辑过去的版本。

**为什么要版本管理：**
- **可重现性** — 将会话固定到已知良好的配置：`{type: "agent", id, version: 3}`
- **安全迭代** — 更新代理而不中断运行在旧版本上的会话
- **回滚** — 如果新系统提示词导致退步，在调试期间将新会话固定回旧版本

**`version` 是可选的。** 省略它（或使用字符串简写 `agent="agent_abc123"`）将在会话创建时获取最新版本。显式传入（`{type: "agent", id, version: N}`）以固定版本确保可重现性。

**获取要固定的版本：** `agents.create()` 和 `agents.update()` 都会在响应中返回 `version`。将其与 `agent_id` 一起保存。要获取现有代理的当前最新版本：`GET /v1/agents/{id}` → `.version`。

**何时更新与新建：** 当代理从概念上仍是同一个代理只是行为有调整时（更好的提示词、额外的工具），使用更新（`POST /v1/agents/{id}`）。当它是不同的角色/用途时，创建新代理。经验法则：如果您会给它相同的 `name`，就更新。

### 代理端点

| 操作        | 方法   | 路径                                  |
| ---------------- | -------- | ------------------------------------- |
| 创建           | `POST`   | `/v1/agents`                          |
| 列表             | `GET`    | `/v1/agents`                          |
| 获取              | `GET`    | `/v1/agents/{id}`                     |
| 更新           | `POST`   | `/v1/agents/{id}`                     |
| 存档          | `POST`   | `/v1/agents/{id}/archive`             |

### 在会话中使用代理

通过字符串 ID（最新版本）或带有显式版本的对象引用代理：

```python
# 字符串简写 — 使用代理的最新版本
session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=environment_id,
)

# 或固定到特定版本（整数）
session = client.beta.sessions.create(
    agent={"type": "agent", "id": agent.id, "version": agent.version},
    environment_id=environment_id,
)
```
