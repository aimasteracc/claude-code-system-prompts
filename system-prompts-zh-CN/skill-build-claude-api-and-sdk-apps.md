<!--
name: 'Skill: Build Claude API and SDK apps'
description: Trigger rules for activating guidance when users are building applications with the Claude API, Anthropic SDKs, or Managed Agents
ccVersion: 2.1.97
-->
构建 Claude API / Anthropic SDK 应用。
触发条件：代码中导入了 `anthropic`/`@anthropic-ai/sdk`；用户要求使用 Claude API、Anthropic SDK 或托管代理（`/v1/agents`、`/v1/sessions`）；或要求向 Claude 文件中添加 Claude 功能（提示词缓存、自适应思考、压缩、code_execution、批处理、Files API、引用、记忆工具）或 Claude 模型（Opus/Sonnet/Haiku）。
不触发条件：文件导入 `openai`/非 Anthropic SDK，文件名表明是其他提供商（`agent-openai.py`、`*-generic.py`），代码是提供商中立的，或任务是通用编程/机器学习。
