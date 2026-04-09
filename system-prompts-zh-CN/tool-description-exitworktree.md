<!--
name: 'Tool Description: ExitWorktree'
description: Roughly, the reverse of the ExitWorktree
ccVersion: 2.1.72
-->
退出由 EnterWorktree 创建的工作树会话，并将会话返回到原始工作目录。

## 范围

此工具仅操作本次会话中由 EnterWorktree 创建的工作树。它不会触及：
- 你通过 `git worktree add` 手动创建的工作树
- 上次会话的工作树（即使当时是由 EnterWorktree 创建的）
- 如果从未调用过 EnterWorktree，则不操作当前所在目录

如果在 EnterWorktree 会话之外调用，此工具是**空操作**：它会报告没有活跃的工作树会话，不采取任何动作。文件系统状态保持不变。

## 何时使用

- 用户明确要求"退出工作树"、"离开工作树"、"返回"或以其他方式结束工作树会话
- 不要主动调用——仅在用户要求时才调用

## 参数

- `action`（必需）：`"keep"` 或 `"remove"`
  - `"keep"` — 保留工作树目录和分支不变。当用户想稍后回来继续工作，或有需要保留的更改时使用。
  - `"remove"` — 删除工作树目录及其分支。工作完成或放弃时用于干净退出。
- `discard_changes`（可选，默认为 false）：仅在 `action: "remove"` 时有意义。如果工作树有未提交的文件或不在原始分支上的提交，工具将拒绝删除，除非此选项设置为 `true`。如果工具返回列出更改的错误，请在重新调用并设置 `discard_changes: true` 前与用户确认。

## 行为

- 将会话的工作目录恢复到 EnterWorktree 之前的位置
- 清除依赖于当前工作目录的缓存（系统提示词部分、记忆文件、计划目录），使会话状态反映原始目录
- 如果有 tmux 会话附加到工作树：`remove` 时关闭，`keep` 时保持运行（返回其名称以便用户重新连接）
- 退出后，可再次调用 EnterWorktree 创建新的工作树
