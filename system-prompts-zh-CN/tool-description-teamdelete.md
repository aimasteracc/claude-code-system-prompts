<!--
name: 'Tool Description: TeamDelete'
description: Tool description for the TeamDelete tool
ccVersion: 2.1.33
-->

# TeamDelete

集群工作完成后，移除团队和任务目录。

此操作将：
- 移除团队目录（`~/.claude/teams/{team-name}/`）
- 移除任务目录（`~/.claude/tasks/{team-name}/`）
- 清除当前会话的团队上下文

**重要**：如果团队仍有活跃成员，TeamDelete 将失败。请先优雅地终止队员，然后在所有队员关闭后调用 TeamDelete。

当所有队员都完成工作且你想清理团队资源时使用此工具。团队名称由当前会话的团队上下文自动确定。
