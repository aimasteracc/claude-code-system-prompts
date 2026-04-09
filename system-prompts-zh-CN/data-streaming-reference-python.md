<!--
name: 'Data: Streaming reference — Python'
description: Python streaming reference including sync/async streaming and handling different content types
ccVersion: 2.1.78
-->
# 流式传输 — Python

## 快速开始

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### 异步方式

```python
async with async_client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## 处理不同内容类型

Claude 可能返回文本、思考块或工具使用。需对每种类型进行适当处理：

> **Opus 4.6：** 使用 `thinking: {type: "adaptive"}`。在旧版模型上，改用 `thinking: {type: "enabled", budget_tokens: N}`。

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Analyze this problem"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[Thinking...]")
            elif event.content_block.type == "text":
                print("\n[Response:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```

---

## 带工具使用的流式传输

Python 工具运行器目前返回完整消息。如需在工具调用中实现逐词元流式传输，请在手动循环中对单个 API 调用使用流式传输：

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # Continue with tool execution if response.stop_reason == "tool_use"
```

---

## 获取最终消息

```python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # Get full message after streaming
    final_message = stream.get_final_message()
    print(f"\n\nTokens used: {final_message.usage.output_tokens}")
```

---

## 带进度更新的流式传输

```python
def stream_with_progress(client, **kwargs):
    """Stream a response with progress updates."""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[Tokens used: {total_tokens}]")
    return "".join(content_parts)
```

---

## 流式传输中的错误处理

```python
try:
    with client.messages.stream(
        model="{{OPUS_ID}}",
        max_tokens=64000,
        messages=[{"role": "user", "content": "Write a story"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\nConnection lost. Please retry.")
except anthropic.RateLimitError:
    print("\nRate limited. Please wait and retry.")
except anthropic.APIStatusError as e:
    print(f"\nAPI error: {e.status_code}")
```

---

## 流事件类型

| 事件类型              | 说明                        | 触发时机                          |
| --------------------- | --------------------------- | --------------------------------- |
| `message_start`       | 包含消息元数据              | 开始时触发一次                    |
| `content_block_start` | 新内容块开始                | 文本/工具使用块开始时             |
| `content_block_delta` | 内容增量更新                | 每个词元/数据块时                 |
| `content_block_stop`  | 内容块完成                  | 块结束时                          |
| `message_delta`       | 消息级别更新                | 包含 `stop_reason`、用量信息      |
| `message_stop`        | 消息完成                    | 结束时触发一次                    |

## 最佳实践

1. **始终刷新输出** — 使用 `flush=True` 以立即显示词元
2. **处理部分响应** — 流式传输中断时，可能产生不完整内容
3. **追踪词元用量** — `message_delta` 事件包含用量信息
4. **设置超时** — 为应用程序设置适当的超时时间
5. **默认使用流式传输** — 使用 `.get_final_message()` 即可在流式传输时获取完整响应，无需处理单个事件即可获得超时保护
