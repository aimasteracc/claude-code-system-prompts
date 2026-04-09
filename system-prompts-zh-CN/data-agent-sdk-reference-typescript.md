<!--
name: 'Data: Agent SDK reference — TypeScript'
description: TypeScript Agent SDK reference including installation, quick start, custom tools, and hooks
ccVersion: 2.1.83
-->
# 代理 SDK — TypeScript

Claude 代理 SDK 提供更高层次的接口，用于构建具备内置工具、安全特性和代理能力的 AI 代理。

## 安装

```bash
npm install @anthropic-ai/claude-agent-sdk
```

---

## 快速入门

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Explain this codebase",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
```

---

## 内置工具

| 工具      | 描述                          |
| --------- | ----------------------------- |
| Read      | 读取工作区中的文件            |
| Write     | 创建新文件                    |
| Edit      | 对已有文件进行精确编辑        |
| Bash      | 执行 shell 命令               |
| Glob      | 按模式查找文件                |
| Grep      | 按内容搜索文件                |
| WebSearch | 在网络上搜索信息              |
| WebFetch        | 获取并分析网页               |
| AskUserQuestion | 向用户提问以澄清问题          |
| Agent           | 生成子代理                   |

---

## 权限系统

```typescript
for await (const message of query({
  prompt: "Refactor the authentication module",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

权限模式：

- `"default"`：对危险操作进行提示
- `"plan"`：仅计划，不执行
- `"acceptEdits"`：自动接受文件编辑
- `"dontAsk"`：不进行提示 — **拒绝**未预先批准的任何操作（不是自动批准模式）
- `"bypassPermissions"`：跳过所有提示（需要在选项中设置 `allowDangerouslySkipPermissions: true`）

---

## MCP（模型上下文协议）支持

```typescript
for await (const message of query({
  prompt: "Open example.com and describe what you see",
  options: {
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

### 进程内 MCP 工具

您可以使用 `tool()` 和 `createSdkMcpServer` 定义在进程内运行的自定义工具：

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const myTool = tool("my-tool", "Description", { input: z.string() }, async (args) => {
  return { content: [{ type: "text", text: "result" }] };
});

const server = createSdkMcpServer({ name: "my-server", tools: [myTool] });

// 传递给 query
for await (const message of query({
  prompt: "Use my-tool to do something",
  options: { mcpServers: { myServer: server } },
})) {
  if ("result" in message) console.log(message.result);
}
```

---

## 钩子

```typescript
import { query, HookCallback } from "@anthropic-ai/claude-agent-sdk";
import { appendFileSync } from "fs";

const logFileChange: HookCallback = async (input) => {
  const filePath = (input as any).tool_input?.file_path ?? "unknown";
  appendFileSync(
    "./audit.log",
    `${new Date().toISOString()}: modified ${filePath}\n`,
  );
  return {};
};

for await (const message of query({
  prompt: "Refactor utils.py to improve readability",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
    hooks: {
      PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }],
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

工具生命周期事件（`PreToolUse`、`PostToolUse`、`PostToolUseFailure`）的钩子事件输入包含 `agent_id` 和 `agent_type` 字段，允许钩子识别是哪个代理（主代理或子代理）触发了工具调用。

可用钩子事件：`PreToolUse`、`PostToolUse`、`PostToolUseFailure`、`Notification`、`UserPromptSubmit`、`SessionStart`、`SessionEnd`、`Stop`、`SubagentStart`、`SubagentStop`、`PreCompact`、`PermissionRequest`、`Setup`、`TeammateIdle`、`TaskCompleted`、`ConfigChange`、`Elicitation`、`ElicitationResult`、`WorktreeCreate`、`WorktreeRemove`、`InstructionsLoaded`

---

## 常用选项

`query()` 接受顶层 `prompt`（字符串）和 `options` 对象：

```typescript
query({ prompt: "...", options: { ... } })
```

| 选项                                | 类型   | 描述                                                                       |
| ----------------------------------- | ------ | -------------------------------------------------------------------------- |
| `cwd`                               | string | 文件操作的工作目录                                                         |
| `allowedTools`                      | array  | 代理可使用的工具（如 `["Read", "Edit", "Bash"]`）                         |
| `tools`                             | array \| preset | 可用的内置工具（`string[]` 或 `{type:'preset', preset:'claude_code'}`）|
| `disallowedTools`                   | array  | 明确禁止的工具                                                             |
| `permissionMode`                    | string | 处理权限提示的方式                                                         |
| `allowDangerouslySkipPermissions`   | bool   | 必须为 `true` 才能使用 `permissionMode: "bypassPermissions"`               |
| `mcpServers`                        | object | 要连接的 MCP 服务器                                                        |
| `hooks`                             | object | 用于自定义行为的钩子                                                       |
| `systemPrompt`                      | string \| preset | 自定义系统提示（`string` 或 `{type:'preset', preset:'claude_code', append?:string}`）|
| `maxTurns`                          | number | 停止前的最大代理轮次                                                       |
| `maxBudgetUsd`                      | number | 查询的最大预算（美元）                                                     |
| `model`                             | string | 模型 ID（默认：由 CLI 决定）                                               |
| `agents`                            | object | 子代理定义（`Record<string, AgentDefinition>`）                            |
| `outputFormat`                      | object | 结构化输出模式                                                             |
| `thinking`                          | object | 思考/推理控制                                                              |
| `betas`                             | array  | 要启用的测试版功能（如 `["context-1m-2025-08-07"]`）                       |
| `settingSources`                    | array  | 要加载的设置（如 `["project"]`）。默认：无（不加载 CLAUDE.md 文件）        |
| `env`                               | object | 会话的环境变量                                                             |
| `agentProgressSummaries`            | bool   | 在 `task_progress` 事件上启用 AI 定期生成的进度摘要                        |

---

## 子代理

```typescript
for await (const message of query({
  prompt: "Use the code-reviewer agent to review this codebase",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert code reviewer for quality and security reviews.",
        prompt: "Analyze code quality and suggest improvements.",
        tools: ["Read", "Glob", "Grep"],
        // 可选：为子代理定制 skills、mcpServers
      },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
```

---

## 消息类型

```typescript
for await (const message of query({
  prompt: "Find TODO comments",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
    console.log(`Stop reason: ${message.stop_reason}`); // 如 "end_turn"、"tool_use"、"max_tokens"
  } else if (message.type === "system" && message.subtype === "init") {
    const sessionId = message.session_id; // 捕获以供后续恢复
  }
}
```

子代理操作也会触发与任务相关的系统消息：
- `task_started` — 注册子代理任务时触发
- `task_progress` — 带累积使用指标、工具调用次数和持续时间的实时进度更新（启用 `agentProgressSummaries` 选项可通过 `summary` 字段获取 AI 定期生成的摘要）
- `task_notification` — 任务完成通知（包含 `tool_use_id` 用于与原始工具调用关联）

---

## 会话历史

获取过去的会话数据：

```typescript
import { listSessions, getSessionMessages, getSessionInfo } from "@anthropic-ai/claude-agent-sdk";

// 列出所有过去的会话（支持通过 limit/offset 进行分页）
const sessions = await listSessions({ limit: 20, offset: 0 });
for (const session of sessions) {
  console.log(`${session.sessionId}: ${session.cwd} (tag: ${session.tag})`);
}

// 获取单个会话的元数据
const sessionId = sessions[0]?.sessionId;
const info = await getSessionInfo(sessionId);
console.log(info.tag, info.createdAt);

// 获取特定会话的消息（支持通过 limit/offset 进行分页）
const messages = await getSessionMessages(sessionId, { limit: 50, offset: 0 });
for (const msg of messages) {
  console.log(msg);
}
```

### 会话变更

重命名、添加标签或分叉会话：

```typescript
import { renameSession, tagSession, forkSession } from "@anthropic-ai/claude-agent-sdk";

// 重命名会话
await renameSession(sessionId, "My refactoring session");

// 为会话添加标签
await tagSession(sessionId, "experiment");

// 清除标签
await tagSession(sessionId, null);

// 分叉会话 — 从特定节点分支对话
const { sessionId: forkedId } = await forkSession(sessionId);
```

---

## MCP 服务器管理

在运行中的查询上运行时管理 MCP 服务器：

```typescript
// 重新连接已断开的 MCP 服务器
await queryHandle.reconnectMcpServer("my-server");

// 开启/关闭 MCP 服务器
await queryHandle.toggleMcpServer("my-server", false);  // (name, enabled) — 两者均必填

// 获取所有已配置 MCP 服务器的状态 — 返回数组
const statuses: McpServerStatus[] = await queryHandle.mcpServerStatus();
for (const s of statuses) {
  console.log(s.name, s.scope, s.tools.length, s.error);
}
```

---

## 最佳实践

1. **始终指定 allowedTools** — 明确列出代理可使用的工具
2. **设置工作目录** — 文件操作时始终指定 `cwd`
3. **使用适当的权限模式** — 从 `"default"` 开始，仅在需要时升级
4. **处理所有消息类型** — 检查 `result` 属性以获取代理输出
5. **限制 maxTurns** — 用合理的限制防止代理失控
