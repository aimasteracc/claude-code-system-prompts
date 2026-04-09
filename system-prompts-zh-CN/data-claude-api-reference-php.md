<!--
name: 'Data: Claude API reference — PHP'
description: PHP SDK reference
ccVersion: 2.1.83
-->
# Claude API — PHP

> **注意：** PHP SDK 是 Anthropic 官方的 PHP SDK。测试版工具运行器可通过 `$client->beta->messages->toolRunner()` 使用。通过 `StructuredOutputModel` 类支持结构化输出辅助功能。不支持 Agent SDK。支持 Bedrock、Vertex AI 和 Foundry 客户端。

## 安装

```bash
composer require "anthropic-ai/sdk"
```

## 客户端初始化

```php
use Anthropic\Client;

// 使用环境变量中的 API 密钥
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
```

### Amazon Bedrock

```php
use Anthropic\Bedrock;

// 构造函数是私有的——使用静态工厂。从环境变量读取 AWS 凭证。
$client = Bedrock\Client::fromEnvironment(region: 'us-east-1');
```

### Google Vertex AI

```php
use Anthropic\Vertex;

// 构造函数是私有的。参数是 `location`，而非 `region`。
$client = Vertex\Client::fromEnvironment(
    location: 'us-east5',
    projectId: 'my-project-id',
);
```

### Anthropic Foundry

```php
use Anthropic\Foundry;

// 构造函数是私有的。baseUrl 或 resource 是必需的。
$client = Foundry\Client::withCredentials(
    authToken: getenv('ANTHROPIC_FOUNDRY_AUTH_TOKEN'),
    baseUrl: 'https://<resource>.services.ai.azure.com/anthropic',
);
```

---

## 基本消息请求

```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);

// content 是多态块的数组（TextBlock、ToolUseBlock、
// ThinkingBlock）。在不检查块类型的情况下访问 content[0] 上的 ->text，
// 如果第一个块不是 TextBlock（例如启用了扩展思考且首先出现 ThinkingBlock），
// 将会抛出异常。请始终进行检查：
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

如果只需要第一个文本块：

```php
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
        break;
    }
}
```

---

## 流式传输

> **需要 SDK v0.5.0+。** v0.4.0 及更早版本使用单个 `$params` 数组；使用命名参数调用会抛出 `Unknown named parameter $model`。升级方法：`composer require "anthropic-ai/sdk:^0.7"`

```php
use Anthropic\Messages\RawContentBlockDeltaEvent;
use Anthropic\Messages\TextDelta;

$stream = $client->messages->createStream(
    model: '{{OPUS_ID}}',
    maxTokens: 64000,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    if ($event instanceof RawContentBlockDeltaEvent && $event->delta instanceof TextDelta) {
        echo $event->delta->text;
    }
}
```

---

## 工具使用

### 工具运行器（测试版）

**测试版：** PHP SDK 通过 `$client->beta->messages->toolRunner()` 提供工具运行器。使用 `BetaRunnableTool` 定义工具——一个定义数组加上一个 `run` 闭包：

```php
use Anthropic\Lib\Tools\BetaRunnableTool;

$weatherTool = new BetaRunnableTool(
    definition: [
        'name' => 'get_weather',
        'description' => 'Get the current weather for a location.',
        'input_schema' => [
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
    run: function (array $input): string {
        return "The weather in {$input['location']} is sunny and 72°F.";
    },
);

$runner = $client->beta->messages->toolRunner(
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'What is the weather in Paris?']],
    model: '{{OPUS_ID}}',
    tools: [$weatherTool],
);

foreach ($runner as $message) {
    foreach ($message->content as $block) {
        if ($block->type === 'text') {
            echo $block->text;
        }
    }
}
```

### 手动循环

工具以数组形式传递。**SDK 使用驼峰命名键**（`inputSchema`、`toolUseID`、`stopReason`），并在传输时自动映射到 API 的蛇形命名——自 v0.5.0 起。循环模式请参阅[共享工具使用概念](../shared/tool-use-concepts.md)。

```php
use Anthropic\Messages\ToolUseBlock;

$tools = [
    [
        'name' => 'get_weather',
        'description' => 'Get the current weather in a given location',
        'inputSchema' => [  // 驼峰命名，而非 input_schema
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
];

$messages = [['role' => 'user', 'content' => 'What is the weather in SF?']];

$response = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    tools: $tools,
    messages: $messages,
);

while ($response->stopReason === 'tool_use') {  // 驼峰命名属性
    $toolResults = [];
    foreach ($response->content as $block) {
        if ($block instanceof ToolUseBlock) {
            // $block->name  : string               — 用于分发的工具名称
            // $block->input : array<string,mixed>  — 解析后的 JSON 输入
            // $block->id    : string               — 作为 toolUseID 传回
            $result = executeYourTool($block->name, $block->input);
            $toolResults[] = [
                'type' => 'tool_result',
                'toolUseID' => $block->id,  // 驼峰命名，而非 tool_use_id
                'content' => $result,
            ];
        }
    }

    // 追加助手轮次 + 带工具结果的用户轮次
    $messages[] = ['role' => 'assistant', 'content' => $response->content];
    $messages[] = ['role' => 'user', 'content' => $toolResults];

    $response = $client->messages->create(
        model: '{{OPUS_ID}}',
        maxTokens: 16000,
        tools: $tools,
        messages: $messages,
    );
}

// 最终文本响应
foreach ($response->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

`$block->type === 'tool_use'` 也有效；`instanceof ToolUseBlock` 可为 PHPStan 缩小类型范围。


---

## 扩展思考

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 会动态决定何时以及思考多少。

```php
use Anthropic\Messages\ThinkingBlock;

$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    thinking: ['type' => 'adaptive'],
    messages: [
        ['role' => 'user', 'content' => 'Solve: 27 * 453'],
    ],
);

// ThinkingBlock 在 content 中位于 TextBlock 之前
foreach ($message->content as $block) {
    if ($block instanceof ThinkingBlock) {
        echo "Thinking:\n{$block->thinking}\n\n";
        // $block->signature 是不透明字符串——在多轮对话中传回思考块时，
        // 请原样保留
    } elseif ($block->type === 'text') {
        echo "Answer: {$block->text}\n";
    }
}
```

> **已弃用：** `['type' => 'enabled', 'budgetTokens' => N]`（固定预算扩展思考）在 Claude 4.6 上仍然有效，但已弃用。请使用上方的自适应思考。

`$block->type === 'thinking'` 也可用于检查；`instanceof` 可为 PHPStan 缩小类型范围。

---

## 提示词缓存

`system:` 接受文本块数组；在最后一个块上设置 `cacheControl`。数组形式语法（驼峰命名键）是惯用写法。有关放置模式和静默失效审查清单，请参阅 `shared/prompt-caching.md`。

```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    system: [
        ['type' => 'text', 'text' => $longSystemPrompt, 'cacheControl' => ['type' => 'ephemeral']],
    ],
    messages: [['role' => 'user', 'content' => 'Summarize the key points']],
);
```

1 小时 TTL：`'cacheControl' => ['type' => 'ephemeral', 'ttl' => '1h']`。`messages->create(...)` 上也有顶级 `cacheControl:` 参数，会自动放置在最后一个可缓存块上。

通过 `$message->usage->cacheCreationInputTokens` / `$message->usage->cacheReadInputTokens` 验证缓存命中。

---

## 结构化输出

### 使用 StructuredOutputModel（推荐）

定义一个实现 `StructuredOutputModel` 的 PHP 类，并将其作为 `outputConfig` 传入：

```php
use Anthropic\Lib\Contracts\StructuredOutputModel;
use Anthropic\Lib\Concerns\StructuredOutputModelTrait;
use Anthropic\Lib\Attributes\Constrained;

class Person implements StructuredOutputModel
{
    use StructuredOutputModelTrait;

    #[Constrained(description: 'Full name')]
    public string $name;

    public int $age;

    public ?string $email = null;  // nullable = 可选字段
}

$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Generate a profile for Alice, age 30']],
    outputConfig: ['format' => Person::class],
);

$person = $message->parsedOutput();  // Person 实例
echo $person->name;
```

类型从 PHP 类型提示推断。使用 `#[Constrained(description: '...')]` 添加描述。可空属性（`?string`）成为可选字段。

### 原始 Schema

```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Extract: John (john@co.com), Enterprise plan']],
    outputConfig: [
        'format' => [
            'type' => 'json_schema',
            'schema' => [
                'type' => 'object',
                'properties' => [
                    'name' => ['type' => 'string'],
                    'email' => ['type' => 'string'],
                    'plan' => ['type' => 'string'],
                ],
                'required' => ['name', 'email', 'plan'],
                'additionalProperties' => false,
            ],
        ],
    ],
);

// 第一个文本块包含有效的 JSON
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        $data = json_decode($block->text, true);
        break;
    }
}
```

---

## 测试版功能与服务器端工具

**`betas:` 不是 `$client->messages->create()` 上的参数**——它只存在于 beta 命名空间中。将其用于需要显式选择头的功能：

```php
use Anthropic\Beta\Messages\BetaRequestMCPServerURLDefinition;

$response = $client->beta->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    mcpServers: [
        BetaRequestMCPServerURLDefinition::with(
            name: 'my-server',
            url: 'https://example.com/mcp',
        ),
    ],
    betas: ['mcp-client-2025-11-20'],  // 仅在 ->beta->messages 上有效
    messages: [['role' => 'user', 'content' => 'Use the MCP tools']],
);
```

**服务器端工具**（bash、web_search、text_editor、code_execution）已正式发布，两种路径均可使用——非 beta 路径使用 `Anthropic\Messages\ToolBash20250124` / `WebSearchTool20260209` / `ToolTextEditor20250728` / `CodeExecutionTool20260120`，beta 路径使用 `Anthropic\Beta\Messages\BetaToolBash20250124` / `BetaWebSearchTool20260209` / `BetaToolTextEditor20250728` / `BetaCodeExecutionTool20260120`。这些工具无需 `betas:` 头。
