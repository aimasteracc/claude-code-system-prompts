<!--
name: 'Data: Managed Agents events and steering'
description: Reference guide for sending and receiving events on managed agent sessions, including streaming, polling, reconnection, message queuing, interrupts, and event payload details
ccVersion: 2.1.97
-->
# 托管代理 — 事件与引导控制

## 事件

### 发送事件

通过 `POST /v1/sessions/{id}/events` 向会话发送事件。

| 事件类型                | 何时发送                                        |
| ------------------------- | --------------------------------------------------- |
| `user.message`            | 发送用户消息 |
| `user.interrupt`          | 在代理运行时中断它 |
| `user.tool_confirmation`  | 批准/拒绝工具调用（当策略为 `always_ask` 时） |
| `user.custom_tool_result` | 为自定义工具调用提供结果 |

### 接收事件

两种方式：

1. **流式传输（SSE）**：`GET /v1/sessions/{id}/events/stream` — 实时服务器推送事件。**长连接** — 服务器定期发送心跳以保持连接活跃。
2. **轮询**：`GET /v1/sessions/{id}/events` — 分页事件列表（查询参数：`limit` 默认 1000，`page`）。**立即返回** — 这是普通的分页 GET，非长轮询。

所有接收到的事件都携带 `id`、`type` 和 `processed_at`（ISO 8601；如果尚未被代理处理则为 `null`）。

> ⚠️ **稳健的轮询（原始 HTTP）。** 如果您绕过 SDK 并自行编写轮询循环，不要依赖 `requests` 或 `httpx` 超时作为挂钟上限——它们是**每块**读取超时，每次有字节到达时重置。涓流响应（心跳、卡住的分块编码体、行为异常的代理）即使设置了 `timeout=(5, 60)` 或 `httpx.Timeout(120)` 也可能无限期阻塞。两个库都没有内置"总挂钟"超时。设置硬性截止时间的方法：在循环层面追踪 `time.monotonic()`，如果单次请求超过预算则中断/取消（例如通过守护线程，或异步 httpx 的 `asyncio.wait_for()`）。**优先使用 SDK** — `client.beta.sessions.events.stream()` 和 `client.beta.sessions.events.list()` 以合理的方式处理超时和重试。
>
> 如果 `GET /v1/sessions/{id}/events`（分页）在收到响应头后挂起，您可能错误地访问了 `GET /v1/sessions/{id}/events` 或遇到了服务端问题——请报告，不要将其视为客户端配置问题。

### 事件类型（接收）

事件类型使用点符号，按命名空间分组：

| 事件类型 | 说明 |
| --- | --- |
| `agent.message` | 代理文本输出 |
| `agent.thinking` | 深度思考块 |
| `agent.tool_use` | 代理使用了内置工具（`agent_toolset_20260401`） |
| `agent.tool_result` | 内置工具的结果 |
| `agent.mcp_tool_use` | 代理使用了 MCP 工具 |
| `agent.mcp_tool_result` | MCP 工具的结果 |
| `agent.custom_tool_use` | 代理调用了自定义工具——会话进入空闲状态，您用 `user.custom_tool_result` 响应 |
| `agent.thread_context_compacted` | 会话上下文已被压缩 |
| `session.status_idle` | 代理已完成当前任务，正在等待输入。它要么等待通过 `user.message` 继续工作的输入，要么等待 `user.custom_tool_result` 或 `user.tool_confirmation`。附带的 `stop_reason` 包含代理停止工作原因的更多信息。 |
| `session.status_running` | 会话已开始运行，代理正在积极工作。 |
| `session.status_rescheduled` | 在发生可重试错误后，会话正在（重新）调度，等待编排系统接管。 |
| `session.status_terminated` | 会话已终止，进入不可逆且不可用的状态。  |
| `session.error` | 处理过程中发生错误 |
| `span.model_request_start` | 模型推理开始 |
| `span.model_request_end` | 模型推理完成 |

流还会回显用户发送的事件（`user.message`、`user.interrupt`、`user.tool_confirmation`、`user.custom_tool_result`）。

---

## 引导控制模式

通过事件接口驱动会话的实用模式。

### 先开流，后发送

**先打开流，再发送事件。** 流只传递在其打开*之后*发生的事件——它不重播当前状态或历史事件。如果您先发送消息后打开流，早期事件（包括快速状态转换）会作为单个批次到达，您将失去实时响应的能力。

```ts
// ✅ 正确 — 并发流式传输和发送
const [response] = await Promise.all([
  streamEvents(sessionId),   // 打开 SSE 连接
  sendMessage(sessionId, text),
]);

// ❌ 错误 — 流打开前的事件作为单个缓冲批次到达
await sendMessage(sessionId, text);
const response = await streamEvents(sessionId);
```

**获取完整历史，** 使用 `GET /v1/sessions/{id}/events`（分页列表）——流只提供从连接时起的实时事件。

### 断流后重连

**SSE 流不支持重播。** 如果您的连接断开（httpx 读取超时、网络抖动）并重连，您只能获取重连*之后*发出的事件。间隔期间发出的任何事件在流中都已丢失。

**合并模式：** 每次（重）连接时，在消费实时流之前先获取完整事件历史，并按事件 ID 去重：

```python
def connect_with_consolidation(client, session_id):
    # 1. 首先打开 SSE 流
    stream = client.beta.sessions.events.stream(session_id=session_id)

    # 2. 获取历史记录以覆盖间隔期间的事件
    history = client.beta.sessions.events.list(
        session_id=session_id,
    )

    # 3. 先返回历史记录，再返回流 — 按 event.id 去重
    seen = set()
    for ev in history.data:
        seen.add(ev.id)
        yield ev
    for ev in stream:
        if ev.id not in seen:
            seen.add(ev.id)
            yield ev
```

### 消息排队

**您不必等待响应再发送下一条消息。** 用户事件在服务端排队，按顺序处理。这对于用户快速连续发送消息的聊天桥接很有用：

```ts
// 三条消息都进入同一个会话；代理按顺序处理
await sendMessage(sessionId, "Summarize the README");
await sendMessage(sessionId, "Actually also check the CONTRIBUTING guide");
await sendMessage(sessionId, "And compare the two");
// 只需流式一次——代理将三者作为一个连贯的轮次响应
```

事件可以随时发送到会话。无需等待特定会话状态再通过 `client.beta.sessions.events.send()` 入队新事件。

### 中断

`interrupt` 事件会**跳过队列**（超过所有待处理的用户消息），强制会话进入 `idle` 状态。用于"停止"/"算了"/"取消"命令：

```ts
await client.beta.sessions.events.send(sessionId, {
  events: [{ type: 'interrupt' }],
});
```

代理在任务中途停止。它不会将中断视为消息——只是停下来。发送后续 `user` 事件来说明接下来该做什么。

> **注意**：在当前实现中，中断事件的 ID 可能为空。排查问题时，使用 `processed_at` 时间戳配合周围事件的 ID。

### 事件载荷

某些事件除状态变更本身外还携带有用的元数据：

`session.status_idle` — 包含一个 `stop_reason` 字段，详细说明会话停止的原因以及用户需要采取的进一步行动类型。
```json
{
  "id": "sevt_456",
  "processed_at": "2026-04-07T04:27:43.197Z",
  "stop_reason": {
    "event_ids": [
      "sevt_123"
    ],
    "type": "requires_action"
  },
  "type": "status_idle"
}
```

`span.model_request_end` 包含 `model_usage` 字段，用于成本跟踪和效率分析：

```json
{
  "type": "span.model_request_end",
  "id": "sevt_456",
  "is_error": false,
  "model_request_start_id": "sevt_123",
  "model_usage": {
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 6656,
    "input_tokens": 3571,
    "output_tokens": 727
  },
  "processed_at": "2026-04-07T04:11:32.189Z"
}
```

**`agent.thread_context_compacted`** — 在会话历史被摘要以适应上下文窗口时发出。包含 `pre_compaction_tokens` 以便您了解压缩了多少：

```json
{
  "id": "sevt_abc123",
  "processed_at": "2026-03-24T14:05:15.787Z",
  "type": "agent.thread_context_compacted"
}
```

### 存档

会话完成后，存档它以释放资源：

```ts
await client.beta.sessions.archive(sessionId);
```
