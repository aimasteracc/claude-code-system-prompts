<!--
name: 'Data: Claude API reference — cURL'
description: Raw API reference for Claude API for use with cURL or else Raw HTTP
ccVersion: 2.1.83
-->
# Claude API — cURL / 原始 HTTP

当用户需要原始 HTTP 请求，或使用没有官方 SDK 的编程语言时，可参考这些示例。

## 配置

```bash
export ANTHROPIC_API_KEY="your-api-key"
```

---

## 基础消息请求

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```

### 解析响应

使用 `jq` 从 JSON 响应中提取字段。不要使用 `grep`/`sed` —
JSON 字符串可以包含任意字符，正则表达式解析在遇到引号、转义或多行内容时会出错。

```bash
# 捕获响应，然后提取字段
response=$(curl -s https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"{{OPUS_ID}}","max_tokens":16000,"messages":[{"role":"user","content":"Hello"}]}')

# 打印第一个文本块（-r 去掉 JSON 引号）
echo "$response" | jq -r '.content[0].text'

# 读取使用量字段
input_tokens=$(echo "$response" | jq -r '.usage.input_tokens')
output_tokens=$(echo "$response" | jq -r '.usage.output_tokens')

# 读取停止原因（用于工具使用循环）
stop_reason=$(echo "$response" | jq -r '.stop_reason')

# 提取所有文本块（content 是数组；过滤 type=="text" 的条目）
echo "$response" | jq -r '.content[] | select(.type == "text") | .text'
```


---

## 流式传输（SSE）

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 64000,
    "stream": true,
    "messages": [{"role": "user", "content": "Write a haiku"}]
  }'
```

响应是一系列服务器发送事件（Server-Sent Events）：

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
```

---

## 工具使用

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
  }'
```

当 Claude 响应包含 `tool_use` 块时，将结果发送回：

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [
      {"role": "user", "content": "What is the weather in Paris?"},
      {"role": "assistant", "content": [
        {"type": "text", "text": "Let me check the weather."},
        {"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "Paris"}}
      ]},
      {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "72°F and sunny"}
      ]}
    ]
  }'
```

---

## 提示缓存

将 `cache_control` 放置在稳定前缀的最后一个块上。放置模式和静默失效审计清单请参见 `shared/prompt-caching.md`。

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "system": [
      {"type": "text", "text": "<large shared prompt...>", "cache_control": {"type": "ephemeral"}}
    ],
    "messages": [{"role": "user", "content": "Summarize the key points"}]
  }'
```

1 小时 TTL 请使用：`"cache_control": {"type": "ephemeral", "ttl": "1h"}`。请求体上的顶层 `"cache_control"` 会自动放置在最后一个可缓存块上。通过响应的 `usage.cache_creation_input_tokens` / `usage.cache_read_input_tokens` 字段验证缓存命中。

---

## 扩展思考

> **Opus 4.6 和 Sonnet 4.6：** 使用自适应思考。`budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上均已弃用。
> **旧版模型：** 使用 `"type": "enabled"` 配合 `"budget_tokens": N`（必须小于 `max_tokens`，最小值 1024）。

```bash
# Opus 4.6：自适应思考（推荐）
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "thinking": {
      "type": "adaptive"
    },
    "output_config": {
      "effort": "high"
    },
    "messages": [{"role": "user", "content": "Solve this step by step..."}]
  }'
```

---

## 必需请求头

| 请求头              | 值                 | 描述                       |
| ------------------- | ------------------ | -------------------------- |
| `Content-Type`      | `application/json` | 必需                       |
| `x-api-key`         | 您的 API 密钥      | 认证                       |
| `anthropic-version` | `2023-06-01`       | API 版本                   |
| `anthropic-beta`    | 测试版功能 ID      | 测试版功能必需             |
