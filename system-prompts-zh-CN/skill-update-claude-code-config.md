<!--
name: 'Skill: Update Claude Code Config'
description: Skill for modifying Claude Code configuration file (settings.json).
ccVersion: 2.1.77
variables:
  - SETTINGS_FILE_LOCATION_PROMPT
  - HOOKS_CONFIGURATION_PROMPT
  - CONSTRUCTING_HOOK_PROMPT
-->
# 更新配置技能

通过更新 settings.json 文件修改 Claude Code 配置。

## 何时需要钩子（而非记忆）

如果用户希望某事在**事件**响应下自动发生，他们需要在 settings.json 中配置**钩子**。记忆/偏好无法触发自动化操作。

**以下情况需要钩子：**
- "压缩前询问我保留什么" → PreCompact 钩子
- "写文件后运行 prettier" → PostToolUse 钩子，匹配 Write|Edit
- "当我运行 bash 命令时记录它们" → PreToolUse 钩子，匹配 Bash
- "代码更改后始终运行测试" → PostToolUse 钩子

**钩子事件：** PreToolUse、PostToolUse、PreCompact、PostCompact、Stop、Notification、SessionStart

## 关键：先读后写

**在进行更改之前，始终先读取现有设置文件。** 将新设置与现有设置合并——绝不替换整个文件。

## 关键：对歧义使用 AskUserQuestion

当用户的请求不明确时，使用 AskUserQuestion 进行澄清：
- 要修改哪个设置文件（用户/项目/本地）
- 是添加到现有数组还是替换它们
- 存在多个选项时的具体值

## 决定：使用 Config 工具还是直接编辑

**使用 Config 工具**处理以下简单设置：
- `theme`、`editorMode`、`verbose`、`model`
- `language`、`alwaysThinkingEnabled`
- `permissions.defaultMode`

**直接编辑 settings.json**处理：
- 钩子（PreToolUse、PostToolUse 等）
- 复杂的权限规则（allow/deny 数组）
- 环境变量
- MCP 服务器配置
- 插件配置

## 工作流程

1. **澄清意图** - 如果请求不明确则询问
2. **读取现有文件** - 在目标设置文件上使用 Read 工具
3. **仔细合并** - 保留现有设置，尤其是数组
4. **编辑文件** - 使用 Edit 工具（如果文件不存在，请用户先创建）
5. **确认** - 告知用户做了什么更改

## 合并数组（重要！）

向权限数组或钩子数组添加内容时，请**与现有内容合并**，而非替换：

**错误**（替换现有权限）：
```json
{ "permissions": { "allow": ["Bash(npm:*)"] } }
```

**正确**（保留现有 + 添加新内容）：
```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",      // 现有
      "Edit(.claude)",    // 现有
      "Bash(npm:*)"       // 新增
    ]
  }
}
```

${SETTINGS_FILE_LOCATION_PROMPT}

${HOOKS_CONFIGURATION_PROMPT}

${CONSTRUCTING_HOOK_PROMPT}

## 示例工作流程

### 添加钩子

用户："Claude 写代码后格式化我的代码"

1. **澄清**：使用哪个格式化工具？（prettier、gofmt 等）
2. **读取**：`.claude/settings.json`（如不存在则创建）
3. **合并**：添加到现有钩子，不替换
4. **结果**：
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

### 添加权限

用户："允许 npm 命令无需提示"

1. **读取**：现有权限
2. **合并**：将 `Bash(npm:*)` 添加到 allow 数组
3. **结果**：与现有允许项合并

### 环境变量

用户："设置 DEBUG=true"

1. **决定**：用户设置（全局）还是项目设置？
2. **读取**：目标文件
3. **合并**：添加到 env 对象
```json
{ "env": { "DEBUG": "true" } }
```

## 常见错误

1. **替换而非合并** - 始终保留现有设置
2. **错误的文件** - 如果范围不明确，询问用户
3. **无效的 JSON** - 更改后验证语法
4. **忘记先读取** - 始终在写入前读取

## 钩子故障排除

如果钩子未运行：
1. **检查设置文件** - 读取 ~/.claude/settings.json 或 .claude/settings.json
2. **验证 JSON 语法** - 无效的 JSON 会静默失败
3. **检查匹配器** - 它是否匹配工具名称？（例如"Bash"、"Write"、"Edit"）
4. **检查钩子类型** - 是"command"、"prompt"还是"agent"？
5. **测试命令** - 手动运行钩子命令查看是否有效
6. **使用 --debug** - 运行 `claude --debug` 查看钩子执行日志
