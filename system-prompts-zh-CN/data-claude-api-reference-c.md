<!--
name: 'Data: Claude API reference — C#'
description: C# SDK reference including installation, client initialization, basic requests, streaming, and tool use
ccVersion: 2.1.83
-->
# Claude API — C#

> **注意：** C# SDK 是 Anthropic 官方的 C# SDK。工具使用通过 Messages API 支持。基于类注解的工具运行器不可用；请使用带有 JSON schema 的原始工具定义。SDK 还支持 Microsoft.Extensions.AI IChatClient 集成与函数调用。

## 安装

```bash
dotnet add package Anthropic
```

## 客户端初始化

```csharp
using Anthropic;

// 默认（使用 ANTHROPIC_API_KEY 环境变量）
AnthropicClient client = new();

// 显式 API 密钥（使用环境变量——切勿硬编码密钥）
AnthropicClient client = new() {
    ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
};
```

---

## 基本消息请求

```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    Messages = [new() { Role = Role.User, Content = "What is the capital of France?" }]
};
var response = await client.Messages.Create(parameters);

// ContentBlock 是联合包装器。.Value 解包为具体变体对象，
// 然后 OfType<T> 过滤所需类型。或使用下方思考部分展示的 TryPick* 模式。
foreach (var text in response.Content.Select(b => b.Value).OfType<TextBlock>())
{
    Console.WriteLine(text.Text);
}
```

---

## 流式传输

```csharp
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 64000,
    Messages = [new() { Role = Role.User, Content = "Write a haiku" }]
};

await foreach (RawMessageStreamEvent streamEvent in client.Messages.CreateStreaming(parameters))
{
    if (streamEvent.TryPickContentBlockDelta(out var delta) &&
        delta.Delta.TryPickText(out var text))
    {
        Console.Write(text.Text);
    }
}
```

**`RawMessageStreamEvent` TryPick 方法**（命名去掉了 `Message`/`Raw` 前缀）：`TryPickStart`、`TryPickDelta`、`TryPickStop`、`TryPickContentBlockStart`、`TryPickContentBlockDelta`、`TryPickContentBlockStop`。没有 `TryPickMessageStop`——请使用 `TryPickStop`。

---

## 思考

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 会动态决定何时以及思考多少。

```csharp
using Anthropic.Models.Messages;

var response = await client.Messages.Create(new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    // ThinkingConfigParam? 从具体变体类隐式转换——无需包装器。
    Thinking = new ThinkingConfigAdaptive(),
    Messages =
    [
        new() { Role = Role.User, Content = "Solve: 27 * 453" },
    ],
});

// ThinkingBlock 在 Content 中位于 TextBlock 之前。TryPick* 缩小联合范围。
foreach (var block in response.Content)
{
    if (block.TryPickThinking(out ThinkingBlock? t))
    {
        Console.WriteLine($"[thinking] {t.Thinking}");
    }
    else if (block.TryPickText(out TextBlock? text))
    {
        Console.WriteLine(text.Text);
    }
}
```

> **已弃用：** `new ThinkingConfigEnabled { BudgetTokens = N }`（固定预算扩展思考）在 Claude 4.6 上仍然有效，但已弃用。请使用上方的自适应思考。

`TryPick*` 的替代方案：`.Select(b => b.Value).OfType<ThinkingBlock>()`（与基本消息示例中的 LINQ 模式相同）。

---

## 工具使用

### 定义工具

`Tool`（而非 `ToolParam`）搭配 `InputSchema` 记录。`InputSchema.Type` 由构造函数自动设置为 `"object"`——无需手动设置。`ToolUnion` 对 `Tool` 有隐式转换，由集合表达式 `[...]` 触发。

```csharp
using System.Text.Json;
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeSonnet4_6,
    MaxTokens = 16000,
    Tools = [
        new Tool {
            Name = "get_weather",
            Description = "Get the current weather in a given location",
            InputSchema = new() {
                Properties = new Dictionary<string, JsonElement> {
                    ["location"] = JsonSerializer.SerializeToElement(
                        new { type = "string", description = "City name" }),
                },
                Required = ["location"],
            },
        },
    ],
    Messages = [new() { Role = Role.User, Content = "Weather in Paris?" }],
};
```

派生自 `anthropic-sdk-csharp/src/Anthropic/Models/Messages/Tool.cs` 和 `ToolUnion.cs:799`（隐式转换）。

循环模式请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。
### 将响应内容转换为后续助手消息

在助手轮次中回显 Claude 响应时，**没有 `.ToParam()` 辅助方法**——需要手动将每个 `ContentBlock` 变体重建为其对应的 `*Param` 类型。不要使用 `new ContentBlockParam(block.Json)`：它可以编译和序列化，但 `.Value` 保持 `null`，因此 `TryPick*`/`Validate()` 会失败（降级为 JSON 透传，而非类型化路径）。

```csharp
using Anthropic.Models.Messages;

Message response = await client.Messages.Create(parameters);

// 无 .ToParam()——按变体逐一重建。每种 *Param 类型到 ContentBlockParam 的
// 隐式转换意味着无需显式包装器。
List<ContentBlockParam> assistantContent = [];
List<ContentBlockParam> toolResults = [];
foreach (ContentBlock block in response.Content)
{
    if (block.TryPickText(out TextBlock? text))
    {
        assistantContent.Add(new TextBlockParam { Text = text.Text });
    }
    else if (block.TryPickThinking(out ThinkingBlock? thinking))
    {
        // 签名必须保留——API 会拒绝被篡改的内容
        assistantContent.Add(new ThinkingBlockParam
        {
            Thinking = thinking.Thinking,
            Signature = thinking.Signature,
        });
    }
    else if (block.TryPickRedactedThinking(out RedactedThinkingBlock? redacted))
    {
        assistantContent.Add(new RedactedThinkingBlockParam { Data = redacted.Data });
    }
    else if (block.TryPickToolUse(out ToolUseBlock? toolUse))
    {
        // ToolUseBlock 有必需的 Caller；ToolUseBlockParam.Caller 是可选的——不要复制它
        assistantContent.Add(new ToolUseBlockParam
        {
            ID = toolUse.ID,
            Name = toolUse.Name,
            Input = toolUse.Input,
        });
        // 执行工具；每个 tool_use 块收集一个结果——如果某个 tool_use ID
        // 没有对应的 tool_result，API 会拒绝后续请求。
        string result = ExecuteYourTool(toolUse.Name, toolUse.Input);
        toolResults.Add(new ToolResultBlockParam
        {
            ToolUseID = toolUse.ID,
            Content = result,
        });
    }
}

// 后续：之前的消息 + 助手回显 + 用户 tool_result
List<MessageParam> followUpMessages =
[
    .. parameters.Messages,
    new() { Role = Role.Assistant, Content = assistantContent },
    new() { Role = Role.User, Content = toolResults },
];
```

`ToolResultBlockParam` 没有元组构造函数——使用对象初始化器。`Content` 是字符串或列表的联合；纯 `string` 可隐式转换。

---

## 上下文编辑/压缩（测试版）

**测试版命名空间前缀不一致**（来源已对 `src/Anthropic/Models/Beta/Messages/*.cs` @ 12.9.0 进行验证）。无前缀：`MessageCreateParams`、`MessageCountTokensParams`、`Role`。**其他所有类型均有 `Beta` 前缀**：`BetaMessageParam`、`BetaMessage`、`BetaContentBlock`、`BetaToolUseBlock`，以及所有块参数类型。无前缀的 `Role` 若与 `Anthropic.Models.Messages.Role` 同时导入，**会**发生 CS0104 冲突。最安全的做法：仅导入 Beta；如需混用，为 beta `Role` 创建别名：

```csharp
using Anthropic.Models.Beta.Messages;
using NonBeta = Anthropic.Models.Messages;  // 仅当需要非 beta 类型时
// 现在：MessageCreateParams, BetaMessageParam, Role (beta 的), NonBeta.Role (如需)
```


`BetaMessage.Content` 是 `IReadOnlyList<BetaContentBlock>`——一个 15 变体的判别联合。使用 `TryPick*` 缩小范围。**响应 `BetaContentBlock` 不可赋值给参数 `BetaContentBlockParam`**——C# 中没有 `.ToParam()`。通过转换每个块来实现往返：

```csharp
using Anthropic.Models.Beta.Messages;

var betaParams = new MessageCreateParams   // 无 Beta 前缀——仅两个无前缀类型之一
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 16000,
    Betas = ["compact-2026-01-12"],
    ContextManagement = new BetaContextManagementConfig
    {
        Edits = [new BetaCompact20260112Edit()],
    },
    Messages = messages,
};
BetaMessage resp = await client.Beta.Messages.Create(betaParams);

foreach (BetaContentBlock block in resp.Content)
{
    if (block.TryPickCompaction(out BetaCompactionBlock? compaction))
    {
        // Content 可为 null——压缩可能在服务器端失败
        Console.WriteLine($"compaction summary: {compaction.Content}");
    }
}

// 上下文编辑元数据位于单独的可空字段上
if (resp.ContextManagement is { } ctx)
{
    foreach (var edit in ctx.AppliedEdits)
        Console.WriteLine($"cleared {edit.ClearedInputTokens} tokens");
}

// 往返：BetaMessageParam.Content 是 BetaMessageParamContent（字符串|列表联合）。
// 它从 List<BetaContentBlockParam> 隐式转换，而非从响应的
// IReadOnlyList<BetaContentBlock>。逐块转换：
List<BetaContentBlockParam> paramBlocks = [];
foreach (var b in resp.Content)
{
    if (b.TryPickText(out var t)) paramBlocks.Add(new BetaTextBlockParam { Text = t.Text });
    else if (b.TryPickCompaction(out var c)) paramBlocks.Add(new BetaCompactionBlockParam { Content = c.Content });
    // ... 根据需要处理其他变体
}
messages.Add(new BetaMessageParam { Role = Role.Assistant, Content = paramBlocks });
```

所有 15 个 `BetaContentBlock.TryPick*` 变体：`Text`、`Thinking`、`RedactedThinking`、`ToolUse`、`ServerToolUse`、`WebSearchToolResult`、`WebFetchToolResult`、`CodeExecutionToolResult`、`BashCodeExecutionToolResult`、`TextEditorCodeExecutionToolResult`、`ToolSearchToolResult`、`McpToolUse`、`McpToolResult`、`ContainerUpload`、`Compaction`。

**`BetaToolUseBlock.Input` 是 `IReadOnlyDictionary<string, JsonElement>`**——按键索引后调用 `JsonElement` 提取器：

```csharp
if (block.TryPickToolUse(out BetaToolUseBlock? tu))
{
    int a = tu.Input["a"].GetInt32();
    string s = tu.Input["name"].GetString()!;
}
```

---

## Effort 参数

Effort 嵌套在 `OutputConfig` 下，**不是**顶级属性。`ApiEnum<string, Effort>` 对枚举有隐式转换，因此可以直接赋值 `Effort.High`。

```csharp
OutputConfig = new OutputConfig { Effort = Effort.High },
```

可选值：`Effort.Low`、`Effort.Medium`、`Effort.High`、`Effort.Max`。与 `Thinking = new ThinkingConfigAdaptive()` 组合使用可实现成本与质量的平衡控制。

---

## 提示词缓存

`System` 接受 `MessageCreateParamsSystem?`——字符串或 `List<TextBlockParam>` 的联合。没有 `SystemTextBlockParam`；使用普通的 `TextBlockParam`。隐式转换需要具体的 `List<TextBlockParam>` 类型（数组字面量不会转换）。有关放置模式和静默失效审查清单，请参阅 `shared/prompt-caching.md`。

```csharp
System = new List<TextBlockParam> {
    new() {
        Text = longSystemPrompt,
        CacheControl = new CacheControlEphemeral(),  // 自动设置 Type = "ephemeral"
    },
},
```

`CacheControlEphemeral` 上的可选 `Ttl`：`new() { Ttl = Ttl.Ttl1h }` 或 `Ttl.Ttl5m`。`CacheControl` 也存在于 `Tool.CacheControl` 和顶级 `MessageCreateParams.CacheControl` 上。

通过 `response.Usage.CacheCreationInputTokens` / `response.Usage.CacheReadInputTokens` 验证缓存命中。

---

## 词元计数

```csharp
MessageTokensCount result = await client.Messages.CountTokens(new MessageCountTokensParams {
    Model = Model.ClaudeOpus4_6,
    Messages = [new() { Role = Role.User, Content = "Hello" }],
});
long tokens = result.InputTokens;
```

`MessageCountTokensParams.Tools` 使用与 `MessageCreateParams.Tools`（`ToolUnion`）不同的联合类型（`MessageCountTokensTool`）——如果传递工具，编译器会在需要时告知。

---

## 结构化输出

```csharp
OutputConfig = new OutputConfig {
    Format = new JsonOutputFormat {
        Schema = new Dictionary<string, JsonElement> {
            ["type"] = JsonSerializer.SerializeToElement("object"),
            ["properties"] = JsonSerializer.SerializeToElement(
                new { name = new { type = "string" } }),
            ["required"] = JsonSerializer.SerializeToElement(new[] { "name" }),
        },
    },
},
```

`JsonOutputFormat.Type` 由构造函数自动设置为 `"json_schema"`。`Schema` 是必需的。

---

## PDF / 文档输入

`DocumentBlockParam` 接受 `DocumentBlockParamSource` 联合：`Base64PdfSource` / `UrlPdfSource` / `PlainTextSource` / `ContentBlockSource`。`Base64PdfSource` 自动设置 `MediaType = "application/pdf"` 和 `Type = "base64"`。

```csharp
new MessageParam {
    Role = Role.User,
    Content = new List<ContentBlockParam> {
        new DocumentBlockParam { Source = new Base64PdfSource { Data = base64String } },
        new TextBlockParam { Text = "Summarize this PDF" },
    },
}
```

---

## 服务器端工具

网页搜索、bash、文本编辑器和代码执行是内置服务器工具。类型名称带有版本后缀；构造函数自动设置 `name`/`type`。所有工具均可隐式转换为 `ToolUnion`。

```csharp
Tools = [
    new WebSearchTool20260209(),
    new ToolBash20250124(),
    new ToolTextEditor20250728(),
    new CodeExecutionTool20260120(),
],
```

另有：`WebFetchTool20260209`、`MemoryTool20250818`。`WebSearchTool20260209` 可选项：`AllowedDomains`、`BlockedDomains`、`MaxUses`、`UserLocation`。

---

## Files API（测试版）

文件位于 `client.Beta.Files` 下（命名空间 `Anthropic.Models.Beta.Files`）。`BinaryContent` 可从 `Stream` 和 `byte[]` 隐式转换。

```csharp
using Anthropic.Models.Beta.Files;
using Anthropic.Models.Beta.Messages;

FileMetadata meta = await client.Beta.Files.Upload(
    new FileUploadParams { File = File.OpenRead("doc.pdf") });

// 引用上传的文件需要 Beta 消息类型：
new BetaRequestDocumentBlock {
    Source = new BetaFileDocumentSource { FileID = meta.ID },
}
```

非 beta 的 `DocumentBlockParamSource` 联合没有文件 ID 变体——文件引用需要 `client.Beta.Messages.Create()`。
