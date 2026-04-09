<!--
name: 'Data: Tool use concepts'
description: Conceptual foundations of tool use with the Claude API including tool definitions, tool choice, and best practices
ccVersion: 2.1.91
-->
# 工具使用概念

本文件涵盖使用 Claude API 进行工具使用的概念基础。有关特定语言的代码示例，请参阅 `python/`、`typescript/` 或其他语言文件夹。有关公开哪些工具、如何在长时间运行的代理中管理上下文以及缓存策略的决策启发式，请参阅 `agent-design.md`。

## 用户自定义工具

### 工具定义结构

> **注意：** 使用工具运行器（测试版）时，工具 schema 会从你的函数签名（Python）、Zod schema（TypeScript）、注解类（Java）、`jsonschema` 结构体标签（Go）或 `BaseTool` 子类（Ruby）自动生成。下面的原始 JSON schema 格式适用于手动方式——包括 PHP 的 `BetaRunnableTool`（将运行闭包包装在手写 schema 周围）——或不支持工具运行器的 SDK。

每个工具需要名称、描述和输入的 JSON Schema：

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g., San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```

**工具定义最佳实践：**

- 使用清晰、描述性的名称（如 `get_weather`、`search_database`、`send_email`）
- 编写详细的描述——Claude 使用这些描述来决定何时使用工具
- 为每个属性包含描述
- 对固定值集合的参数使用 `enum`
- 在 `required` 中标记真正必需的参数；其他参数设为可选并提供默认值

---

### 工具选择选项

控制 Claude 何时使用工具：

| 值                                | 行为                                          |
| --------------------------------- | --------------------------------------------- |
| `{"type": "auto"}`                | Claude 决定是否使用工具（默认）               |
| `{"type": "any"}`                 | Claude 必须使用至少一个工具                   |
| `{"type": "tool", "name": "..."}` | Claude 必须使用指定的工具                     |
| `{"type": "none"}`                | Claude 不能使用工具                           |

任何 `tool_choice` 值都可以包含 `"disable_parallel_tool_use": true`，以强制 Claude 每次响应最多使用一个工具。默认情况下，Claude 可能在单次响应中请求多个工具调用。

---

### 工具运行器与手动循环

**工具运行器（推荐）：** SDK 的工具运行器自动处理代理循环——它调用 API、检测工具使用请求、执行你的工具函数、将结果反馈给 Claude，并重复直到 Claude 停止调用工具。在 Python、TypeScript、Java、Go、Ruby 和 PHP SDK 中可用（测试版）。Python SDK 还提供 MCP 转换辅助工具（`anthropic.lib.tools.mcp`），用于将 MCP 工具、提示词和资源转换为工具运行器可用的格式——详情见 `python/claude-api/tool-use.md`。

**手动代理循环：** 当需要对循环进行细粒度控制时使用（如自定义日志记录、条件工具执行、人机协作审批）。循环直到 `stop_reason == "end_turn"`，始终追加完整的 `response.content` 以保留 tool_use 块，并确保每个 `tool_result` 包含匹配的 `tool_use_id`。

**服务器端工具的停止原因：** 使用服务器端工具（代码执行、网页搜索等）时，API 运行服务器端采样循环。如果此循环达到默认的 10 次迭代限制，响应的 `stop_reason` 将为 `"pause_turn"`。要继续，重新发送用户消息和助手响应，然后再次发起 API 请求——服务器将从中断处恢复。**不要**添加额外的用户消息如"继续"——API 检测到尾部的 `server_tool_use` 块后会自动知道要恢复。

```python
# 在代理循环中处理 pause_turn
if response.stop_reason == "pause_turn":
    messages = [
        {"role": "user", "content": user_query},
        {"role": "assistant", "content": response.content},
    ]
    # 再次发起 API 请求——服务器自动恢复
    response = client.messages.create(
        model="{{OPUS_ID}}", messages=messages, tools=tools
    )
```

设置 `max_continuations` 限制（如 5）以防止无限循环。完整指南见：`https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons`

> **安全性：** 工具运行器在 Claude 请求时自动执行你的工具函数。对于有副作用的工具（发送邮件、修改数据库、金融交易），在工具函数内验证输入，并考虑对破坏性操作要求确认。如果需要在每次工具执行前进行人工审批，请使用手动代理循环。

---

### 处理工具结果

当 Claude 使用工具时，响应包含一个 `tool_use` 块。你必须：

1. 用提供的输入执行工具
2. 在 `tool_result` 消息中发送结果
3. 继续对话

**工具结果中的错误处理：** 当工具执行失败时，将 `"is_error": true` 并提供具有信息量的错误消息。Claude 通常会确认错误，然后尝试不同的方法或请求澄清。

**多个工具调用：** Claude 可以在单次响应中请求多个工具。在继续之前处理所有工具——在单个 `user` 消息中发送所有结果。

---

## 服务器端工具：代码执行

代码执行工具让 Claude 在安全的沙箱容器中运行代码。与用户自定义工具不同，服务器端工具在 Anthropic 的基础设施上运行——你不需要在客户端执行任何操作。只需包含工具定义，Claude 处理其余部分。

### 关键事实

- 在隔离容器中运行（1 CPU，5 GiB RAM，5 GiB 磁盘）
- 无互联网访问（完全沙箱化）
- 预装了数据科学库的 Python 3.11
- 容器持续 30 天，可跨请求重用
- 与网页搜索/网页抓取工具一起使用时免费；否则每月每组织前 1,550 小时免费，之后 $0.05/小时

### 工具定义

工具不需要 schema——只需在 `tools` 数组中声明它：

```json
{
  "type": "code_execution_20260120",
  "name": "code_execution"
}
```

Claude 自动获得对 `bash_code_execution`（运行 shell 命令）和 `text_editor_code_execution`（创建/查看/编辑文件）的访问权限。

### 预安装的 Python 库

- **数据科学**：pandas、numpy、scipy、scikit-learn、statsmodels
- **可视化**：matplotlib、seaborn
- **文件处理**：openpyxl、xlsxwriter、pillow、pypdf、pdfplumber、python-docx、python-pptx
- **数学**：sympy、mpmath
- **实用工具**：tqdm、python-dateutil、pytz、sqlite3

可在运行时通过 `pip install` 安装额外包。

### 支持上传的文件类型

| 类型   | 扩展名                             |
| ------ | ---------------------------------- |
| 数据   | CSV、Excel (.xlsx/.xls)、JSON、XML |
| 图像   | JPEG、PNG、GIF、WebP               |
| 文本   | .txt、.md、.py、.js 等             |

### 容器重用

跨请求重用容器以维持状态（文件、已安装包、变量）。从第一次响应中提取 `container_id`，并将其传递给后续请求。

### 响应结构

响应包含交错的文本和工具结果块：

- `text` — Claude 的解释
- `server_tool_use` — Claude 正在执行的操作
- `bash_code_execution_tool_result` — 代码执行输出（检查 `return_code` 判断成功/失败）
- `text_editor_code_execution_tool_result` — 文件操作结果

> **安全性：** 在将下载的文件写入磁盘之前，始终使用 `os.path.basename()` / `path.basename()` 清理文件名，以防止路径遍历攻击。将文件写入专用的输出目录。

---

## 服务器端工具：网页搜索和网页抓取

网页搜索和网页抓取让 Claude 搜索网络并检索页面内容。它们在服务器端运行——只需包含工具定义，Claude 自动处理查询、抓取和结果处理。

### 工具定义

```json
[
  { "type": "web_search_20260209", "name": "web_search" },
  { "type": "web_fetch_20260209", "name": "web_fetch" }
]
```

### 动态过滤（Opus 4.6 / Sonnet 4.6）

`web_search_20260209` 和 `web_fetch_20260209` 版本支持**动态过滤**——Claude 编写并执行代码，在搜索结果进入上下文窗口之前对其进行过滤，提高准确性和词元效率。动态过滤内置于这些工具版本中，自动激活；你无需单独声明 `code_execution` 工具或传递任何测试版头。

```json
{
  "tools": [
    { "type": "web_search_20260209", "name": "web_search" },
    { "type": "web_fetch_20260209", "name": "web_fetch" }
  ]
}
```

没有动态过滤时，也可使用之前的 `web_search_20250305` 版本。

> **注意：** 仅在你的应用程序需要代码执行来完成其自身目的（数据分析、文件处理、可视化）时（独立于网页搜索），才包含独立的 `code_execution` 工具。将其与 `_20260209` 网页工具一起包含会创建第二个执行环境，可能会让模型感到困惑。

---

## 服务器端工具：程序化工具调用

使用标准工具调用时，每次工具调用都是一次往返：Claude 调用，结果进入 Claude 的上下文，Claude 推理，然后调用下一个工具。链式调用会累积延迟和词元——大多数中间数据不再需要。

程序化工具调用让 Claude 将这些调用组合成一个脚本。脚本在代码执行容器中运行；当它调用工具时，容器暂停，调用执行，结果返回到正在运行的代码（而非 Claude 的上下文）。脚本用普通控制流处理它。只有最终输出返回给 Claude。当链接大量工具调用或中间结果较大且在进入上下文窗口前应过滤时，使用此方法。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling`

---

## 服务器端工具：工具搜索

工具搜索工具让 Claude 从大型库中动态发现工具，无需将所有定义加载到上下文窗口中。当你有许多工具但任何给定请求只有少数相关时使用它。发现的工具 schema 被追加到请求中，而不是换入——这保留了提示词缓存（参阅 `agent-design.md` 中的缓存部分）。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool`

---

## 技能

技能将特定任务的指令打包，Claude 仅在相关时加载。每个技能是一个包含 `SKILL.md` 文件的文件夹。技能的简短描述默认保留在上下文中；当当前任务需要时，Claude 读取完整文件。使用技能将专业指令保留在基础系统提示词之外，同时不失可发现性。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/skills`

---

## 工具使用示例

你可以在工具定义中直接提供工具调用示例，以演示使用模式并减少参数错误。这有助于 Claude 理解如何正确格式化工具输入，特别是对于具有复杂 schema 的工具。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use`

---

## 服务器端工具：计算机使用

计算机使用让 Claude 与桌面环境交互（截图、鼠标、键盘）。它可以是 Anthropic 托管的（服务器端，类似代码执行）或自托管的（你提供环境并在客户端执行操作）。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview`

---

## 上下文编辑

上下文编辑在长时间运行的代理积累轮次时，从记录中清除陈旧的工具结果和思考块。与压缩（汇总）不同，上下文编辑是修剪——清除的内容被删除，而非替换。当旧的工具输出不再相关，且你希望保持记录精简而不丢失对话结构时使用它。可配置清除内容的阈值。

完整文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/build-with-claude/context-editing`

---

## 客户端工具：记忆

记忆工具使 Claude 能够通过记忆文件目录跨对话存储和检索信息。Claude 可以创建、读取、更新和删除在会话之间持久存在的文件。

### 关键事实

- 客户端工具——你通过实现控制存储
- 支持命令：`view`、`create`、`str_replace`、`insert`、`delete`、`rename`
- 在 `/memories` 目录中的文件上操作
- Python、TypeScript 和 Java SDK 提供辅助类/函数用于实现记忆后端

> **安全性：** 切勿在记忆文件中存储 API 密钥、密码、令牌或其他机密。对个人身份信息（PII）要谨慎——在持久化用户数据之前检查数据隐私法规（GDPR、CCPA）。参考实现没有内置访问控制；在多用户系统中，在工具处理程序中实现每用户记忆目录和认证。

完整实现示例，请使用 WebFetch：

- 文档：`https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md`

---

## 结构化输出

结构化输出约束 Claude 的响应遵循特定的 JSON schema，保证输出有效且可解析。这不是一个单独的工具——它增强了 Messages API 响应格式和/或工具参数验证。

有两种功能可用：

- **JSON 输出**（`output_config.format`）：控制 Claude 的响应格式
- **严格工具使用**（`strict: true`）：保证工具参数 schema 有效

**支持的模型：** {{OPUS_NAME}}、{{SONNET_NAME}} 和 {{HAIKU_NAME}}。旧版模型（Claude Opus 4.5、Claude Opus 4.1）也支持结构化输出。

> **推荐：** 使用 `client.messages.parse()`，它会自动根据你的 schema 验证响应。直接使用 `messages.create()` 时，使用 `output_config: {format: {...}}`。`output_format` 便捷参数也被某些 SDK 方法接受（如 `.parse()`），但 `output_config.format` 是规范的 API 级参数。

### JSON Schema 限制

**支持：**

- 基本类型：object、array、string、integer、number、boolean、null
- `enum`、`const`、`anyOf`、`allOf`、`$ref`/`$def`
- 字符串格式：`date-time`、`time`、`date`、`duration`、`email`、`hostname`、`uri`、`ipv4`、`ipv6`、`uuid`
- `additionalProperties: false`（所有对象必需）

**不支持：**

- 递归 schema
- 数值约束（`minimum`、`maximum`、`multipleOf`）
- 字符串约束（`minLength`、`maxLength`）
- 复杂数组约束
- `additionalProperties` 设置为 `false` 以外的值

Python 和 TypeScript SDK 自动处理不支持的约束，方法是从发送到 API 的 schema 中删除它们，并在客户端验证。

### 重要说明

- **首次请求延迟**：新 schema 会产生一次性编译成本。使用相同 schema 的后续请求使用 24 小时缓存。
- **拒绝**：如果 Claude 因安全原因拒绝（`stop_reason: "refusal"`），输出可能不符合你的 schema。
- **词元限制**：如果 `stop_reason: "max_tokens"`，输出可能不完整。增大 `max_tokens`。
- **不兼容**：引文（返回 400 错误）、消息预填充。
- **兼容**：批次 API、流式传输、词元计数、扩展思考。

---

## 有效工具使用的技巧

1. **提供详细描述**：Claude 非常依赖描述来理解何时以及如何使用工具
2. **使用具体的工具名称**：`get_current_weather` 优于 `weather`
3. **验证输入**：在执行之前始终验证工具输入
4. **优雅地处理错误**：返回信息丰富的错误消息，以便 Claude 可以适应
5. **限制工具数量**：工具过多会让模型困惑——保持集合精简
6. **测试工具交互**：验证 Claude 在各种场景中正确使用工具

详细的工具使用文档，请使用 WebFetch：

- URL: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview`
