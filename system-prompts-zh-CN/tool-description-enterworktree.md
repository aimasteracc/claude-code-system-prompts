<!--
name: 'Tool Description: EnterWorktree'
description: Tool description for the EnterWorktree tool.
ccVersion: 2.1.72
-->
仅在用户明确要求在工作树中工作时才使用此工具。此工具创建一个隔离的 git 工作树，并将当前会话切换到其中。

## 何时使用

- 用户明确提到"工作树"（例如"开始一个工作树"、"在工作树中工作"、"创建一个工作树"、"使用工作树"）

## 何时不使用

- 用户要求创建分支、切换分支或在其他分支上工作——请改用 git 命令
- 用户要求修复错误或开发功能——除非他们明确提到工作树，否则使用普通 git 工作流
- 除非用户明确提到"工作树"，否则绝不使用此工具

## 要求

- 必须在 git 仓库中，或者在 settings.json 中配置了 WorktreeCreate/WorktreeRemove 钩子
- 不能已经在工作树中

## 行为

- 在 git 仓库中：在 `.claude/worktrees/` 内创建一个新 git 工作树，基于 HEAD 创建新分支
- 在 git 仓库外：委托给 WorktreeCreate/WorktreeRemove 钩子实现与版本控制系统无关的隔离
- 将会话的工作目录切换到新工作树
- 使用 ExitWorktree 可在会话中途离开工作树（保留或删除）。会话退出时如果仍在工作树中，将提示用户选择保留或删除

## 参数

- `name`（可选）：工作树的名称。如不提供，将自动生成随机名称。
