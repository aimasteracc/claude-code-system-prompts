<!--
name: 'Data: GitHub App installation PR description'
description: Template for PR description when installing Claude Code GitHub App integration
ccVersion: 2.0.14
-->
## 🤖 安装 Claude Code GitHub App

此 PR 添加了一个 GitHub Actions 工作流，在我们的仓库中启用 Claude Code 集成。

### 什么是 Claude Code？

[Claude Code](https://claude.com/claude-code) 是一个 AI 编程代理，可协助完成以下工作：
- 修复 Bug 和改进代码
- 更新文档
- 实现新功能
- 代码审查与建议
- 编写测试
- 以及更多！

### 工作原理

此 PR 合并后，我们可以通过在拉取请求或 Issue 评论中提及 @claude 来与其交互。
工作流触发后，Claude 将分析评论及其周围上下文，并在 GitHub Action 中执行请求的任务。

### 重要说明

- **此工作流在 PR 合并前不会生效**
- **@claude 提及在合并完成前不会起作用**
- 每当 Claude 在 PR 或 Issue 评论中被提及时，工作流将自动触发
- Claude 可访问完整的 PR 或 Issue 上下文，包括文件、差异和之前的评论

### 安全性

- 我们的 Anthropic API 密钥作为 GitHub Actions 密钥安全存储
- 只有对仓库具有写入权限的用户才能触发工作流
- 所有 Claude 运行记录都保存在 GitHub Actions 运行历史中
- Claude 的默认工具仅限于读写文件，以及通过创建评论、分支和提交与我们的仓库交互
- 可通过在工作流文件中添加以下内容来允许更多工具：

```
allowed_tools: Bash(npm install),Bash(npm run build),Bash(npm run lint),Bash(npm run test)
```

更多信息请参阅 [Claude Code action 仓库](https://github.com/anthropics/claude-code-action)。

合并此 PR 后，欢迎在任意 PR 的评论中提及 @claude 来开始使用！
