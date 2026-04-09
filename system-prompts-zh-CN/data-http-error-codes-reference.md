<!--
name: 'Data: HTTP error codes reference'
description: Reference for HTTP error codes returned by the Claude API with common causes and handling strategies
ccVersion: 2.1.73
-->
# HTTP 错误码参考

本文档说明 Claude API 返回的 HTTP 错误码、常见原因及处理方式。如需特定语言的错误处理示例，请参阅 `python/` 或 `typescript/` 文件夹。

## 错误码概览

| 错误码 | 错误类型                | 可重试 | 常见原因                         |
| ------ | ----------------------- | ------ | -------------------------------- |
| 400    | `invalid_request_error` | 否     | 请求格式或参数无效               |
| 401    | `authentication_error`  | 否     | API 密钥无效或缺失               |
| 403    | `permission_error`      | 否     | API 密钥权限不足                 |
| 404    | `not_found_error`       | 否     | 端点或模型 ID 无效               |
| 413    | `request_too_large`     | 否     | 请求超出大小限制                 |
| 429    | `rate_limit_error`      | 是     | 请求过于频繁                     |
| 500    | `api_error`             | 是     | Anthropic 服务问题               |
| 529    | `overloaded_error`      | 是     | API 暂时过载                     |

## 详细错误信息

### 400 Bad Request（请求错误）

**原因：**

- 请求体中的 JSON 格式错误
- 缺少必需参数（`model`、`max_tokens`、`messages`）
- 参数类型无效（如需要整数却传入了字符串）
- 消息数组为空
- 消息角色未交替出现（user/assistant）

**错误示例：**

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

**修复方法：** 发送前验证请求结构，确认：

- `model` 为有效模型 ID
- `max_tokens` 为正整数
- `messages` 数组非空且角色交替正确

---

### 401 Unauthorized（未授权）

**原因：**

- 缺少 `x-api-key` 请求头或 `Authorization` 请求头
- API 密钥格式无效
- API 密钥已撤销或删除

**修复方法：** 确保 `ANTHROPIC_API_KEY` 环境变量已正确设置。

---

### 403 Forbidden（禁止访问）

**原因：**

- API 密钥无权访问所请求的模型
- 组织级别的限制
- 尝试访问未获授权的测试版功能

**修复方法：** 在控制台检查 API 密钥权限。您可能需要使用不同的 API 密钥，或申请访问特定功能的权限。

---

### 404 Not Found（未找到）

**原因：**

- 模型 ID 拼写错误（如 `claude-sonnet-4.6` 而非 `claude-sonnet-4-6`）
- 使用了已弃用的模型 ID
- API 端点无效

**修复方法：** 使用模型文档中的精确模型 ID。可以使用别名（如 `{{OPUS_ID}}`）。

---

### 413 Request Too Large（请求体过大）

**原因：**

- 请求体超过最大大小限制
- 输入词元过多
- 图片数据过大

**修复方法：** 减小输入大小——截断对话历史、压缩/缩小图片，或将大文档拆分为多个片段。

---

### 400 参数验证错误

部分 400 错误专门与参数验证相关：

- `max_tokens` 超出模型限制
- `temperature` 值无效（必须在 0.0-1.0 之间）
- 扩展思考中 `budget_tokens` >= `max_tokens`
- 工具定义 schema 无效

**扩展思考的常见错误：**

```
# Wrong: budget_tokens must be < max_tokens
thinking: budget_tokens=10000, max_tokens=1000  → Error!

# Correct
thinking: budget_tokens=10000, max_tokens=16000
```

---

### 429 Rate Limited（频率限制）

**原因：**

- 超出每分钟请求数（RPM）
- 超出每分钟词元数（TPM）
- 超出每天词元数（TPD）

**需检查的响应头：**

- `retry-after`：重试前等待的秒数
- `x-ratelimit-limit-*`：您的限额
- `x-ratelimit-remaining-*`：剩余配额

**修复方法：** Anthropic SDK 会自动以指数退避方式重试 429 和 5xx 错误（默认：`max_retries=2`）。如需自定义重试行为，请参阅对应语言的错误处理示例。

---

### 500 Internal Server Error（内部服务器错误）

**原因：**

- Anthropic 服务临时故障
- API 处理出现 Bug

**修复方法：** 以指数退避方式重试。如持续出现，请检查 [status.anthropic.com](https://status.anthropic.com)。

---

### 529 Overloaded（过载）

**原因：**

- API 需求量高
- 服务容量已满

**修复方法：** 以指数退避方式重试。考虑使用其他模型（Haiku 通常负载较低）、将请求分散到更长时间段，或实现请求队列。

---

## 常见错误及修复方法

| 错误操作                        | 错误码          | 修复方法                                                   |
| ------------------------------- | --------------- | ---------------------------------------------------------- |
| `budget_tokens` >= `max_tokens` | 400             | 确保 `budget_tokens` < `max_tokens`                        |
| 模型 ID 拼写错误                | 404             | 使用有效的模型 ID，如 `{{OPUS_ID}}`                        |
| 第一条消息角色为 `assistant`    | 400             | 第一条消息必须为 `user`                                    |
| 连续相同角色的消息              | 400             | 交替使用 `user` 和 `assistant`                             |
| API 密钥写在代码中              | 401（密钥泄漏） | 使用环境变量                                               |
| 需要自定义重试                  | 429/5xx         | SDK 自动重试；通过 `max_retries` 自定义                    |

## SDK 中的类型化异常

**务必使用 SDK 的类型化异常类**，而非通过字符串匹配检查错误消息。每个 HTTP 错误码对应一个特定的异常类：

| HTTP 错误码 | TypeScript 类                     | Python 类                         |
| ----------- | --------------------------------- | --------------------------------- |
| 400         | `Anthropic.BadRequestError`       | `anthropic.BadRequestError`       |
| 401         | `Anthropic.AuthenticationError`   | `anthropic.AuthenticationError`   |
| 403         | `Anthropic.PermissionDeniedError` | `anthropic.PermissionDeniedError` |
| 404         | `Anthropic.NotFoundError`         | `anthropic.NotFoundError`         |
| 429         | `Anthropic.RateLimitError`        | `anthropic.RateLimitError`        |
| 500+        | `Anthropic.InternalServerError`   | `anthropic.InternalServerError`   |
| 任意        | `Anthropic.APIError`              | `anthropic.APIError`              |

```typescript
// ✅ Correct: use typed exceptions
try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.RateLimitError) {
    // Handle rate limiting
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error ${error.status}:`, error.message);
  }
}

// ❌ Wrong: don't check error messages with string matching
try {
  const response = await client.messages.create({...});
} catch (error) {
  const msg = error instanceof Error ? error.message : String(error);
  if (msg.includes("429") || msg.includes("rate_limit")) { ... }
}
```

所有异常类均继承自 `Anthropic.APIError`，该类具有 `status` 属性。使用 `instanceof` 检查时，应从最具体到最宽泛（例如先检查 `RateLimitError`，再检查 `APIError`）。
