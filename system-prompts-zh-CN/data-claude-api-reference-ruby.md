<!--
name: 'Data: Claude API reference — Ruby'
description: Ruby SDK reference including installation, client initialization, basic requests, streaming, and beta tool runner
ccVersion: 2.1.83
-->
# Claude API — Ruby

> **注意：** Ruby SDK 支持 Claude API。工具运行器可通过 `client.beta.messages.tool_runner()` 以测试版形式使用。代理 SDK 暂不支持 Ruby。

## 安装

```bash
gem install anthropic
```

## 客户端初始化

```ruby
require "anthropic"

# 默认（使用 ANTHROPIC_API_KEY 环境变量）
client = Anthropic::Client.new

# 显式指定 API 密钥
client = Anthropic::Client.new(api_key: "your-api-key")
```

---

## 基础消息请求

```ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
# content 是多态块对象（TextBlock、ThinkingBlock、
# ToolUseBlock 等）的数组。.type 是 Symbol — 与 :text 比较，而非 "text"。
# 对非 TextBlock 条目调用 .text 会引发 NoMethodError。
message.content.each do |block|
  puts block.text if block.type == :text
end
```

---

## 流式传输

```ruby
stream = client.messages.stream(
  model: :"{{OPUS_ID}}",
  max_tokens: 64000,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```

---

## 工具使用

Ruby SDK 通过原始 JSON 模式定义支持工具使用，同时提供用于自动执行工具的测试版工具运行器。

### 工具运行器（测试版）

```ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
```

### 手动循环

工具定义格式和代理循环模式请参见[共享工具使用概念](../shared/tool-use-concepts.md)。

---

## 提示缓存

`system_:`（末尾下划线 — 避免遮蔽 `Kernel#system`）接受文本块数组；在最后一个块上设置 `cache_control`。普通哈希通过 `OrHash` 类型别名也可使用。放置模式和静默失效审计清单请参见 `shared/prompt-caching.md`。

```ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 16000,
  system_: [
    { type: "text", text: long_system_prompt, cache_control: { type: "ephemeral" } }
  ],
  messages: [{ role: "user", content: "Summarize the key points" }]
)
```

1 小时 TTL 请使用：`cache_control: { type: "ephemeral", ttl: "1h" }`。`messages.create` 上还有顶层 `cache_control:` 会自动放置在最后一个可缓存块上。

通过 `message.usage.cache_creation_input_tokens` / `message.usage.cache_read_input_tokens` 验证缓存命中。
