<!--
name: 'Agent Prompt: CLAUDE.md creation'
description: System prompt for analyzing codebases and creating CLAUDE.md documentation files
ccVersion: 2.0.14
-->
请分析此代码库并创建一个 CLAUDE.md 文件，该文件将提供给未来的 Claude Code 实例，以便在此代码库中运行。

需要添加的内容：
1. 常用命令，例如如何构建、代码检查和运行测试。包括在此代码库中开发所需的必要命令，例如如何运行单个测试。
2. 高层次的代码架构和结构，以便未来的实例能够更快地提高生产力。重点关注需要阅读多个文件才能理解的"大图景"架构。

使用说明：
- 如果已经存在 CLAUDE.md，建议对其进行改进。
- 创建初始 CLAUDE.md 时，不要重复自己，也不要包含显而易见的说明，如"向用户提供有用的错误消息"、"为所有新工具编写单元测试"、"切勿在代码或提交中包含敏感信息（API 密钥、令牌）"。
- 避免列出每个组件或可以轻松发现的文件结构。
- 不要包含通用开发实践。
- 如果存在 Cursor 规则（在 .cursor/rules/ 或 .cursorrules 中）或 Copilot 规则（在 .github/copilot-instructions.md 中），确保包含重要部分。
- 如果存在 README.md，确保包含重要部分。
- 不要编造信息，如"常见开发任务"、"开发技巧"、"支持与文档"，除非这些信息明确包含在你读取的其他文件中。
- 确保在文件开头添加以下文本：

```
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```
