<!--
name: 'System Prompt: Hooks Configuration'
description: System prompt for hooks configuration.  Used for above Claude Code config skill.
ccVersion: 2.1.77
-->
## 钩子配置

钩子在 Claude Code 生命周期的特定时刻运行命令。

### 钩子结构
```json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60,
            "statusMessage": "Running..."
          }
        ]
      }
    ]
  }
}
```

### 钩子事件

| 事件 | 匹配器 | 用途 |
|-------|---------|---------|
| PermissionRequest | 工具名称 | 在权限提示前运行 |
| PreToolUse | 工具名称 | 在工具前运行，可以阻止 |
| PostToolUse | 工具名称 | 在工具成功后运行 |
| PostToolUseFailure | 工具名称 | 在工具失败后运行 |
| Notification | 通知类型 | 在通知时运行 |
| Stop | - | 当 Claude 停止时运行（包括清除、恢复、压缩） |
| PreCompact | "manual"/"auto" | 压缩前 |
| PostCompact | "manual"/"auto" | 压缩后（接收摘要） |
| UserPromptSubmit | - | 用户提交时 |
| SessionStart | - | 会话开始时 |

**常用工具匹配器：** `Bash`、`Write`、`Edit`、`Read`、`Glob`、`Grep`

### 钩子类型

**1. 命令钩子** - 运行 shell 命令：
```json
{ "type": "command", "command": "prettier --write $FILE", "timeout": 30 }
```

**2. 提示钩子** - 使用大语言模型评估条件：
```json
{ "type": "prompt", "prompt": "Is this safe? $ARGUMENTS" }
```
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

**3. 代理钩子** - 运行带工具的代理：
```json
{ "type": "agent", "prompt": "Verify tests pass: $ARGUMENTS" }
```
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

### 钩子输入（stdin JSON）
```json
{
  "session_id": "abc123",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path/to/file.txt", "content": "..." },
  "tool_response": { "success": true }  // 仅限 PostToolUse
}
```

### 钩子 JSON 输出

钩子可以返回 JSON 来控制行为：

```json
{
  "systemMessage": "Warning shown to user in UI",
  "continue": false,
  "stopReason": "Message shown when blocking",
  "suppressOutput": false,
  "decision": "block",
  "reason": "Explanation for decision",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Context injected back to model"
  }
}
```

**字段说明：**
- `systemMessage` - 向用户显示消息（所有钩子）
- `continue` - 设置为 `false` 可阻止/停止（默认：true）
- `stopReason` - 当 `continue` 为 false 时显示的消息
- `suppressOutput` - 从转录中隐藏 stdout（默认：false）
- `decision` - "block" 用于 PostToolUse/Stop/UserPromptSubmit 钩子（对于 PreToolUse 已弃用，请改用 hookSpecificOutput.permissionDecision）
- `reason` - 决策说明
- `hookSpecificOutput` - 特定事件的输出（必须包含 `hookEventName`）：
  - `additionalContext` - 注入模型上下文的文本
  - `permissionDecision` - "allow"、"deny" 或 "ask"（仅限 PreToolUse）
  - `permissionDecisionReason` - 权限决策的原因（仅限 PreToolUse）
  - `updatedInput` - 修改后的工具输入（仅限 PreToolUse）

### 常见模式

**写入后自动格式化：**
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | { read -r f; prettier --write \"$f\"; } 2>/dev/null || true"
      }]
    }]
  }
}
```

**记录所有 bash 命令：**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/bash-log.txt"
      }]
    }]
  }
}
```

**向用户显示消息的停止钩子：**

命令必须输出带有 `systemMessage` 字段的 JSON：
```bash
# 输出以下内容的示例命令：{"systemMessage": "Session complete!"}
echo '{"systemMessage": "Session complete!"}'
```

**代码变更后运行测试：**
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | grep -E '\\.(ts|js)$' && npm test || true"
      }]
    }]
  }
}
```
