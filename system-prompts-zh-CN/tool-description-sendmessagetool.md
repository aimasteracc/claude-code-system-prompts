<!--
name: 'Tool Description: SendMessageTool'
description: Agent teams version of SendMessageTool.
ccVersion: 2.1.83
-->

# SendMessage

向另一个代理发送消息。

```json
{"to": "researcher", "summary": "分配任务 1", "message": "开始处理任务 #1"}
```

| `to` | |
|---|---|
| `"researcher"` | 按名称指定队员 |
| `"*"` | 广播给所有队员——成本高昂（与团队规模成线性关系），仅在每个人都真正需要时使用 |${""}

你的纯文本输出对其他代理不可见——要进行通信，你必须调用此工具。来自队员的消息会自动传递；你不需要检查收件箱。通过名称而非 UUID 指代队员。转发消息时，不要引用原始消息——它已经呈现给用户了。${""}

## 协议响应（旧版）

如果你收到包含 `type: "shutdown_request"` 或 `type: "plan_approval_request"` 的 JSON 消息，请用对应的 `_response` 类型回复——回显 `request_id`，设置 `approve` 为 true/false：

```json
{"to": "team-lead", "message": {"type": "shutdown_response", "request_id": "...", "approve": true}}
{"to": "researcher", "message": {"type": "plan_approval_response", "request_id": "...", "approve": false, "feedback": "add error handling"}}
```

批准关闭将终止你的进程。拒绝计划会让队员回去修改。除非被要求，否则不要发起 `shutdown_request`。不要发送结构化的 JSON 状态消息——请使用 TaskUpdate。
