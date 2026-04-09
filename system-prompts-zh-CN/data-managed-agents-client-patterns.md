<!--
name: 'Data: Managed Agents client patterns'
description: Reference guide of common client-side patterns for driving Managed Agent sessions, including stream reconnection, idle-break gating, tool confirmations, interrupts, and custom tools
ccVersion: 2.1.97
-->
# 托管代理 — 常见客户端模式

驱动托管代理会话时您需要在客户端编写的模式，基于可运行的 SDK 示例。

代码示例使用 TypeScript——Python 和 cURL 遵循相同结构；等效实现见 `python/managed-agents/README.md` 和 `curl/managed-agents.md`。

---

## 1. 无损流重连

**问题：** SSE 不支持重播。如果连接在会话中途断开，简单的重连会从"现在"重新打开流，期间发出的所有事件将被静默丢失。

**解决方案：** 重连时，在消费实时流之前先通过 `events.list()` 获取完整事件历史，并在实时流赶上时按事件 ID 去重。

```ts
const seenEventIds = new Set<string>()
const stream = await client.beta.sessions.events.stream(session.id)

// 流现在已打开并在服务端缓冲。先读取历史记录。
for await (const event of client.beta.sessions.events.list(session.id)) {
  seenEventIds.add(event.id)
  handle(event)
}

// 跟踪实时流。去重只限制 handle()——终止检查必须
// 即使对已见事件也运行，否则历史响应中的终止事件
// 会被 `continue` 跳过，循环永不退出。
for await (const event of stream) {
  if (!seenEventIds.has(event.id)) {
    seenEventIds.add(event.id)
    handle(event)
  }
  if (event.type === 'session.status_terminated') break
  if (event.type === 'session.status_idle' && event.stop_reason.type !== 'requires_action') break
}
```

---

## 2. `processed_at` — 排队 vs 已处理

流上的每个事件都携带 `processed_at`（ISO 8601）。对于客户端发送的事件（`user.message`、`user.interrupt`、`user.tool_confirmation`、`user.custom_tool_result`），当事件已排队但尚未被代理接取时为 `null`，代理处理后才会填充时间戳。同一事件在流上出现两次——一次 `processed_at: null`，一次带有时间戳。

```ts
for await (const event of stream) {
  if (event.type === 'user.message') {
    if (event.processed_at == null) onQueued(event.id)
    else onProcessed(event.id, event.processed_at)
  }
}
```

用此驱动您发送内容的"待处理 → 已确认"UI 状态。如何将本地渲染的乐观消息映射到服务器分配的 `event.id` 是应用程序特定的（通常通过 `events.send()` 的返回值或 FIFO 排序）。

---

## 3. 中断运行中的会话

将 `user.interrupt` 作为普通事件发送。会话继续运行直到到达安全边界，然后进入空闲状态。

```ts
await client.beta.sessions.events.send(session.id, {
  events: [{ type: 'user.interrupt' }],
})

// 排空直到会话真正完成——完整的门控逻辑见模式 5。
for await (const event of stream) {
  if (event.type === 'session.status_terminated') break
  if (
    event.type === 'session.status_idle' &&
    event.stop_reason.type !== 'requires_action'
  ) break
}
```

参考：`interrupt.ts`——一看到 `span.model_request_start` 就立即发送中断，排空到空闲，然后通过 `sessions.retrieve()` 验证。

---

## 4. `tool_confirmation` 往返

当代理有 `permission_policy: { type: 'always_ask' }` 时，对该工具的任何调用都会触发 `evaluated_permission === 'ask'` 的 `agent.tool_use` 事件，会话进入空闲等待决策。用 `user.tool_confirmation` 响应。

```ts
for await (const event of stream) {
  if (event.type === 'agent.tool_use' && event.evaluated_permission === 'ask') {
    await client.beta.sessions.events.send(session.id, {
      events: [{
        type: 'user.tool_confirmation',
        tool_use_id: event.id,         // 不是 toolu_ ID——使用 event.id
        result: 'allow',               // 或 'deny'
        // deny_message: '...',        // 可选，仅在 result: 'deny' 时使用
      }],
    })
  }
}
```

关键点：
- `tool_use_id` 是 `event.id`（通常是 `sevt_...`），**不是** `toolu_...` ID。
- `result` 是 `'allow' | 'deny'`。使用 `deny_message` 告知模型*为何*拒绝——它会被反馈给代理。
- 多个待处理工具：对每个 `evaluated_permission === 'ask'` 的 `agent.tool_use` 事件响应一次。

参考：`tool-permissions.ts`。

---

## 5. 正确的空闲中断门控

不要仅在 `session.status_idle` 时中断。会话会短暂进入空闲——例如在并行工具执行之间、等待 `user.tool_confirmation` 或等待 `user.custom_tool_result` 时。在空闲状态且 `stop_reason` 为终止时中断，或在 `session.status_terminated` 时中断。

```ts
for await (const event of stream) {
  handle(event)
  if (event.type === 'session.status_terminated') break
  if (event.type === 'session.status_idle') {
    if (event.stop_reason.type === 'requires_action') continue // 等待您处理——不要中断
    break // end_turn 或 retries_exhausted——两者都是终止状态
  }
}
```

`session.status_idle` 上的 `stop_reason.type` 值：
- `requires_action` — 代理正在等待客户端事件（工具确认、自定义工具结果）。处理它，不要中断。
- `retries_exhausted` — 终止失败。中断后检查 `sessions.retrieve()` 获取错误状态。
- `end_turn` — 正常完成。

---

## 6. 空闲后状态写入竞争

SSE 流在会话的可查询状态反映变化之前略早发出 `session.status_idle`。在空闲时中断并立即调用 `sessions.delete()` 或 `sessions.archive()` 的客户端会间歇性地收到 400 错误"cannot delete/archive while running"（运行时无法删除/存档）。

清理前轮询：

```ts
let s
for (let i = 0; i < 10; i++) {
  s = await client.beta.sessions.retrieve(session.id)
  if (s.status !== 'running') break
  await new Promise(r => setTimeout(r, 200))
}
if (s?.status !== 'running') {
  await client.beta.sessions.archive(session.id)
} // 否则：2 秒后仍在运行——不要存档，等待其稳定或升级处理
```

---

## 7. 先开流，后发送

始终在发送启动事件**之前**打开流。否则代理可能在您的消费者附加之前处理事件并发出最初的事件，您将错过它们。

```ts
const stream = await client.beta.sessions.events.stream(session.id)
await client.beta.sessions.events.send(session.id, {
  events: [{ type: 'user.message', content: [{ type: 'text', text: 'Hello' }] }],
})
for await (const event of stream) { /* ... */ }
```

`Promise.all([stream, send])` 形式也有效，但先开流更简单且效果相同——流在打开的瞬间就开始缓冲。

---

## 8. 文件挂载注意事项

**挂载的资源有与您上传的文件不同的 `file_id`。** 会话创建会制作一个会话范围的副本。

```ts
const uploaded = await client.beta.files.upload({ file, purpose: 'agent_resource' })
// uploaded.id         → 原始文件
const session = await client.beta.sessions.create({
  /* ... */
  resources: [{ type: 'file', file_id: uploaded.id, mount_path: '/workspace/data.csv' }],
})
// session.resources[0].file_id !== uploaded.id  ← 不同的 ID
```

通过 `files.delete(uploaded.id)` 删除原始文件；会话范围的副本会随会话一起垃圾回收。`mount_path` 必须是绝对路径——见 `shared/managed-agents-environments.md`。

---

## 9. 通过自定义工具将凭据保留在主机端

**问题：** 将第三方 API 密钥放入代理的保险库或环境意味着沙箱持有该密钥。对于绑定到某个人的密钥（Linear 个人密钥、`gh` CLI 认证）或您不想传入容器的密钥，这是不可取的。

**解决方案：** 将操作暴露为自定义工具。代理发出 `agent.custom_tool_use`；您的编排器使用自己的凭据执行调用，并用 `user.custom_tool_result` 响应。容器永远看不到密钥。

```ts
// 代理模板：声明工具，无凭据
tools: [{ type: 'custom', name: 'linear_graphql', input_schema: { /* query, vars */ } }]

// 编排器：使用主机端凭据处理调用
for await (const event of stream) {
  if (event.type === 'agent.custom_tool_use' && event.name === 'linear_graphql') {
    const result = await linear.request(event.input.query, event.input.vars) // 主机的密钥
    await client.beta.sessions.events.send(session.id, {
      events: [{ type: 'user.custom_tool_result', tool_use_id: event.id, result }],
    })
  }
}
```

同样的模式适用于 `gh` CLI、本地评估脚本，或任何需要主机专用认证或二进制文件的场景。
