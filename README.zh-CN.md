<div>
<div align="right">
<a href="https://piebald.ai"><img width="200" top="20" align="right" src="https://github.com/Piebald-AI/.github/raw/main/Wordmark.svg"></a>
</div>

<div align="left">

### 了解 Piebald
我们发布了 **Piebald**，终极代理式 AI 开发者体验。\
立即下载并免费试用！**https://piebald.ai/**

<a href="https://piebald.ai/discord"><img src="https://img.shields.io/badge/Join%20our%20Discord-5865F2?style=flat&logo=discord&logoColor=white" alt="加入我们的 Discord"></a>
<a href="https://x.com/PiebaldAI"><img src="https://img.shields.io/badge/Follow%20%40PiebaldAI-000000?style=flat&logo=x&logoColor=white" alt="X"></a>

<sub>[**向下滚动查看 Claude Code 的系统提示词。**](#claude-code-系统提示词) :point_down:</sub>

</div>
</div>

<div align="left">
<a href="https://piebald.ai">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://piebald.ai/screenshot-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://piebald.ai/screenshot-light.png">
  <img alt="主图" width="800" src="https://piebald.ai/screenshot-light.png">
</picture>
</a>
</div>

# Claude Code 系统提示词

[![在 Awesome Claude Code 中被提及](https://awesome.re/mentioned-badge.svg)](https://github.com/hesreallyhim/awesome-claude-code)

> [!important]
> **最新动态（2026 年 1 月 23 日）：我们已将 Claude Code 的约 40 条系统提醒全部添加到此列表中——请参阅[系统提醒](#系统提醒)。**

本仓库包含截至 **[Claude Code v2.1.92](https://www.npmjs.com/package/@anthropic-ai/claude-code/v/2.1.92)（2026 年 4 月 3 日）** 的所有 Claude Code 各类系统提示词及其对应词元数的最新列表。同时还包含一份 [**CHANGELOG.md**](./CHANGELOG.md)，记录了自 v2.0.14 以来 141 个版本中系统提示词的变更历史。来自 [<img src="https://github.com/Piebald-AI/piebald/raw/main/assets/logo.svg" width="15"> **Piebald**](https://piebald.ai/) 团队。

**本仓库在每次 Claude Code 发布后数分钟内即更新。请参阅[变更日志](./CHANGELOG.md)，并在 X 上关注 [@PiebaldAI](https://x.com/PiebaldAI) 以获取每次发布的系统提示词变更摘要。**

> [!note]
> ⭐ **Star** 本仓库以获取 Claude Code 新版本通知。每发布一个新版本，我们都会在 GitHub 上创建一个 Release，所有已 Star 仓库的用户都会收到通知。

---

为什么有多个"系统提示词"？

**Claude Code 的系统提示词并非单一字符串。**

实际上，它由以下部分组成：
- 根据环境和各种配置条件性添加的大段内容。
- 内置工具（如 `Write`、`Bash`、`TodoWrite`）的描述，其中部分描述相当庞大。
- 内置代理（如 Explore 和 Plan）的独立系统提示词。
- 众多 AI 驱动的实用功能，如会话压缩、`CLAUDE.md` 生成、会话标题生成等，各有其独立的系统提示词。

最终结果——110 余个字符串，在一个庞大的混淆 JS 文件中不断变化和移动。

> [!TIP]
> 想要在自己的 Claude Code 安装中**修改特定的系统提示词部分**？**请使用 [tweakcc](https://github.com/Piebald-AI/tweakcc)。** 它——
> - 允许你以 Markdown 文件的形式自定义系统提示词的各个部分，然后
> - 将你的基于 npm 或原生（二进制）的 Claude Code 安装打上补丁，并且
> - 在你与 Anthropic 对同一提示词文件存在冲突修改时提供差异比对和冲突管理。

## 提取方式

本仓库中的系统提示词是使用脚本从最新 npm 版本的 Claude Code 中提取的。由于这些提示词直接从 Claude Code 的编译源代码中提取，因此可以保证与 Claude Code 实际使用的内容完全一致。如果你使用 [tweakcc](https://github.com/Piebald-AI/tweakcc) 自定义系统提示词，其工作方式类似——它会在你的本地安装中为与本仓库提取的相同字符串打补丁。

## 提示词列表

请注意，部分提示词包含插值内容，例如内置工具名称引用、可用子代理列表以及各种上下文特定变量，因此在特定 Claude Code 会话中的实际词元数会略有不同——但偏差通常不超过 ±20 tks。

### 代理提示词

子代理与实用工具。

#### 子代理

- [代理提示词：Explore](./system-prompts/agent-prompt-explore.md)（**494** tks）- Explore 子代理的系统提示词。
- [代理提示词：计划模式（增强版）](./system-prompts/agent-prompt-plan-mode-enhanced.md)（**636** tks）- Plan 子代理的增强版提示词。

#### 创建助手

- [代理提示词：代理创建架构师](./system-prompts/agent-prompt-agent-creation-architect.md)（**1110** tks）- 用于创建具有详细规范的自定义 AI 代理的系统提示词。
- [代理提示词：CLAUDE.md 创建](./system-prompts/agent-prompt-claudemd-creation.md)（**384** tks）- 用于分析代码库并创建 CLAUDE.md 文档文件的系统提示词。
- [代理提示词：状态行设置](./system-prompts/agent-prompt-status-line-setup.md)（**1999** tks）- 用于配置状态行显示的 statusline-setup 代理的系统提示词。

#### 斜杠命令

- [代理提示词：/batch 斜杠命令](./system-prompts/agent-prompt-batch-slash-command.md)（**1106** tks）- 用于在代码库中编排大规模可并行化变更的指令。
- [代理提示词：/review-pr 斜杠命令](./system-prompts/agent-prompt-review-pr-slash-command.md)（**211** tks）- 用于通过代码分析审查 GitHub 拉取请求（PR）的系统提示词。
- [代理提示词：/schedule 斜杠命令](./system-prompts/agent-prompt-schedule-slash-command.md)（**2486** tks）- 引导用户通过 Anthropic 云 API 以 cron 触发器调度、更新、列出或运行远程 Claude Code 代理。
- [代理提示词：/security-review 斜杠命令](./system-prompts/agent-prompt-security-review-slash-command.md)（**2607** tks）- 全面的安全审查提示词，用于分析代码变更并聚焦于可利用漏洞。

#### 实用工具

- [代理提示词：代理钩子](./system-prompts/agent-prompt-agent-hook.md)（**133** tks）- "代理钩子"的提示词。
- [代理提示词：自动模式规则审查器](./system-prompts/agent-prompt-auto-mode-rule-reviewer.md)（**257** tks）- 审查并评估用户定义的自动模式分类规则的清晰度、完整性、冲突和可操作性。
- [代理提示词：Bash 命令描述生成器](./system-prompts/agent-prompt-bash-command-description-writer.md)（**207** tks）- 为 bash 命令生成清晰简洁的主动语态命令描述的指令。
- [代理提示词：Bash 命令前缀检测](./system-prompts/agent-prompt-bash-command-prefix-detection.md)（**823** tks）- 用于检测命令前缀和命令注入的系统提示词。
- [代理提示词：Claude 指南代理](./system-prompts/agent-prompt-claude-guide-agent.md)（**734** tks）- 帮助用户理解和有效使用 Claude Code、Claude Agent SDK 及 Claude API 的 claude-guide 代理系统提示词。
- [代理提示词：编程会话标题生成器](./system-prompts/agent-prompt-coding-session-title-generator.md)（**181** tks）- 为编程会话生成标题。
- [代理提示词：会话摘要](./system-prompts/agent-prompt-conversation-summarization.md)（**1121** tks）- 用于创建详细会话摘要的系统提示词。
- [代理提示词：确定要附加的记忆文件](./system-prompts/agent-prompt-determine-which-memory-files-to-attach.md)（**265** tks）- 用于确定要为主代理附加哪些记忆文件的代理。
- [代理提示词：Dream 记忆整合](./system-prompts/agent-prompt-dream-memory-consolidation.md)（**727** tks）- 指示代理执行多阶段记忆整合流程——定位现有记忆、从日志和记录中收集近期信号、将更新合并到主题文件，并精简索引。
- [代理提示词：通用](./system-prompts/agent-prompt-general-purpose.md)（**285** tks）- 在代码库中搜索、分析和编辑代码并向调用方简洁报告发现结果的通用子代理系统提示词。
- [代理提示词：钩子条件评估器（停止）](./system-prompts/agent-prompt-hook-condition-evaluator-stop.md)（**145** tks）- 用于评估 Claude Code 中钩子条件（特别是停止条件）的系统提示词。
- [代理提示词：提示词建议生成器 v2](./system-prompts/agent-prompt-prompt-suggestion-generator-v2.md)（**296** tks）- 为 Claude Code 生成提示词建议的 V2 版指令。
- [代理提示词：快速创建 PR](./system-prompts/agent-prompt-quick-pr-creation.md)（**806** tks）- 使用预填充上下文创建提交和拉取请求（PR）的简化提示词。
- [代理提示词：快速 git 提交](./system-prompts/agent-prompt-quick-git-commit.md)（**510** tks）- 使用预填充上下文创建单个 git 提交的简化提示词。
- [代理提示词：近期消息摘要](./system-prompts/agent-prompt-recent-message-summarization.md)（**724** tks）- 用于对近期消息进行摘要的代理提示词。
- [代理提示词：自主代理行为安全监控器（第一部分）](./system-prompts/agent-prompt-security-monitor-for-autonomous-agent-actions-first-part.md)（**3101** tks）- 指示 Claude 充当安全监控器，根据阻止/允许规则评估自主编码代理行为，以防止提示词注入、范围蔓延和意外损坏。
- [代理提示词：自主代理行为安全监控器（第二部分）](./system-prompts/agent-prompt-security-monitor-for-autonomous-agent-actions-second-part.md)（**3325** tks）- 定义环境上下文、阻止规则和允许例外，以管理代理可以或不可以执行哪些工具操作。
- [代理提示词：会话搜索助手](./system-prompts/agent-prompt-session-search-assistant.md)（**426** tks）- 根据用户查询和元数据查找相关会话的会话搜索助手代理提示词。
- [代理提示词：会话记忆更新指令](./system-prompts/agent-prompt-session-memory-update-instructions.md)（**756** tks）- 在会话期间更新会话记忆文件的指令。
- [代理提示词：会话标题和分支生成](./system-prompts/agent-prompt-session-title-and-branch-generation.md)（**307** tks）- 用于生成简洁会话标题和 git 分支名称的代理。
- [代理提示词：验证专家](./system-prompts/agent-prompt-verification-specialist.md)（**2938** tks）- 一个验证子代理的系统提示词，通过运行构建、测试套件、代码检查器和对抗性探测来对实现进行对抗性测试，然后发布通过/失败/部分通过的裁决。
- [代理提示词：WebFetch 摘要器](./system-prompts/agent-prompt-webfetch-summarizer.md)（**189** tks）- 为主模型摘要 WebFetch 冗长输出的代理提示词。
- [代理提示词：工作进程 fork 执行](./system-prompts/agent-prompt-worker-fork-execution.md)（**404** tks）- 直接执行指令而不派生更多子代理，然后报告结构化结果的 fork 工作子代理系统提示词。

### 数据

嵌入 Claude Code 中各种模板文件的内容。

- [数据：Agent SDK 模式——Python](./system-prompts/data-agent-sdk-patterns-python.md)（**2656** tks）- Python Agent SDK 模式，包括自定义工具、钩子、子代理、MCP 集成和会话恢复。
- [数据：Agent SDK 模式——TypeScript](./system-prompts/data-agent-sdk-patterns-typescript.md)（**1529** tks）- TypeScript Agent SDK 模式，包括基础代理、钩子、子代理和 MCP 集成。
- [数据：Agent SDK 参考——Python](./system-prompts/data-agent-sdk-reference-python.md)（**3299** tks）- Python Agent SDK 参考，包括安装、快速入门、通过 MCP 自定义工具和钩子。
- [数据：Agent SDK 参考——TypeScript](./system-prompts/data-agent-sdk-reference-typescript.md)（**2943** tks）- TypeScript Agent SDK 参考，包括安装、快速入门、自定义工具和钩子。
- [数据：Claude API 参考——C#](./system-prompts/data-claude-api-reference-c.md)（**4341** tks）- C# SDK 参考，包括安装、客户端初始化、基本请求、流式传输和工具使用。
- [数据：Claude API 参考——Go](./system-prompts/data-claude-api-reference-go.md)（**4294** tks）- Go SDK 参考。
- [数据：Claude API 参考——Java](./system-prompts/data-claude-api-reference-java.md)（**4506** tks）- Java SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 beta 工具使用。
- [数据：Claude API 参考——PHP](./system-prompts/data-claude-api-reference-php.md)（**3486** tks）- PHP SDK 参考。
- [数据：Claude API 参考——Python](./system-prompts/data-claude-api-reference-python.md)（**3549** tks）- Python SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮会话。
- [数据：Claude API 参考——Ruby](./system-prompts/data-claude-api-reference-ruby.md)（**923** tks）- Ruby SDK 参考，包括安装、客户端初始化、基本请求、流式传输和 beta 工具运行器。
- [数据：Claude API 参考——TypeScript](./system-prompts/data-claude-api-reference-typescript.md)（**2881** tks）- TypeScript SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮会话。
- [数据：Claude API 参考——cURL](./system-prompts/data-claude-api-reference-curl.md)（**2174** tks）- 用于 cURL 或原始 HTTP 的 Claude API 原始接口参考。
- [数据：Claude 模型目录](./system-prompts/data-claude-model-catalog.md)（**2295** tks）- 当前和历史 Claude 模型的目录，包含精确的模型 ID、别名、上下文窗口和定价。
- [数据：Files API 参考——Python](./system-prompts/data-files-api-reference-python.md)（**1334** tks）- Python Files API 参考，包括文件上传、列出、删除和在消息中使用。
- [数据：Files API 参考——TypeScript](./system-prompts/data-files-api-reference-typescript.md)（**797** tks）- TypeScript Files API 参考，包括文件上传、列出、删除和在消息中使用。
- [数据：@claude 提及的 GitHub Actions 工作流](./system-prompts/data-github-actions-workflow-for-claude-mentions.md)（**527** tks）- 通过 @claude 提及触发 Claude Code 的 GitHub Actions 工作流模板。
- [数据：GitHub App 安装 PR 描述](./system-prompts/data-github-app-installation-pr-description.md)（**424** tks）- 安装 Claude Code GitHub App 集成时的拉取请求（PR）描述模板。
- [数据：HTTP 错误码参考](./system-prompts/data-http-error-codes-reference.md)（**1922** tks）- Claude API 返回的 HTTP 错误码参考，包含常见原因和处理策略。
- [数据：实时文档来源](./system-prompts/data-live-documentation-sources.md)（**2629** tks）- 用于从官方来源获取当前 Claude API 和 Agent SDK 文档的 WebFetch URL。
- [数据：Message Batches API 参考——Python](./system-prompts/data-message-batches-api-reference-python.md)（**1544** tks）- Python Batches API 参考，包括批次创建、状态轮询和以 50% 费用检索结果。
- [数据：提示词缓存——设计与优化](./system-prompts/data-prompt-caching-design-optimization.md)（**2657** tks）- 关于如何设计提示词构建代码以实现有效缓存的文档，包括放置模式和反模式。
- [数据：会话记忆模板](./system-prompts/data-session-memory-template.md)（**292** tks）- 会话记忆 `summary.md` 文件的模板结构。
- [数据：流式传输参考——Python](./system-prompts/data-streaming-reference-python.md)（**1528** tks）- Python 流式传输参考，包括同步/异步流式传输和处理不同内容类型。
- [数据：流式传输参考——TypeScript](./system-prompts/data-streaming-reference-typescript.md)（**1703** tks）- TypeScript 流式传输参考，包括基本流式传输和处理不同内容类型。
- [数据：工具使用概念](./system-prompts/data-tool-use-concepts.md)（**4139** tks）- Claude API 工具使用的概念基础，包括工具定义、工具选择和最佳实践。
- [数据：工具使用参考——Python](./system-prompts/data-tool-use-reference-python.md)（**5106** tks）- Python 工具使用参考，包括工具运行器、手动代理循环、代码执行和结构化输出。
- [数据：工具使用参考——TypeScript](./system-prompts/data-tool-use-reference-typescript.md)（**5033** tks）- TypeScript 工具使用参考，包括工具运行器、手动代理循环、代码执行和结构化输出。

### 系统提示词

主系统提示词的各个部分。

- [系统提示词：Advisor 工具指令](./system-prompts/system-prompt-advisor-tool-instructions.md)（**443** tks）- 使用 Advisor 工具的指令。
- [系统提示词：代理摘要生成](./system-prompts/system-prompt-agent-summary-generation.md)（**178** tks）- 用于"代理摘要"生成的系统提示词。
- [系统提示词：代理记忆指令](./system-prompts/system-prompt-agent-memory-instructions.md)（**337** tks）- 在代理系统提示词中包含记忆更新指导的指令。
- [系统提示词：代理线程注意事项](./system-prompts/system-prompt-agent-thread-notes.md)（**205** tks）- 代理线程的行为准则，涵盖绝对路径、响应格式、避免使用表情符号和工具调用标点符号。
- [系统提示词：自动模式](./system-prompts/system-prompt-auto-mode.md)（**255** tks）- 持续任务执行，类似于后台代理。
- [系统提示词：避免不必要的 sleep 命令（PowerShell 工具描述的一部分）](./system-prompts/system-prompt-avoiding-unnecessary-sleep-commands-part-of-powershell-tool-description.md)（**182** tks）- 在 PowerShell 脚本中避免不必要 sleep 命令的指南，包括等待和通知的替代方案。
- [系统提示词：伙伴模式](./system-prompts/system-prompt-buddy-mode.md)（**205** tks）- 生成驻留在终端中并评论开发者工作的编程伙伴的指令，重点是根据给定统计数据和灵感词汇创建令人难忘的独特个性。
- [系统提示词：屏蔽协助恶意活动](./system-prompts/system-prompt-censoring-assistance-with-malicious-activities.md)（**98** tks）- 在屏蔽恶意活动请求的同时，协助授权安全测试、防御性安全、CTF 挑战和教育场景的指南。
- [系统提示词：Chrome 浏览器 MCP 工具](./system-prompts/system-prompt-chrome-browser-mcp-tools.md)（**156** tks）- 在使用前通过 MCPSearch 加载 Chrome 浏览器 MCP 工具的指令。
- [系统提示词：Claude 在 Chrome 浏览器自动化中的使用](./system-prompts/system-prompt-claude-in-chrome-browser-automation.md)（**759** tks）- 有效使用 Claude 的 Chrome 浏览器自动化工具的指令。
- [系统提示词：上下文压缩摘要](./system-prompts/system-prompt-context-compaction-summary.md)（**278** tks）- 用于上下文压缩摘要（SDK 使用）的提示词。
- [系统提示词：记忆指令的描述部分](./system-prompts/system-prompt-description-part-of-memory-instructions.md)（**148** tks）- 描述记忆_内容_的字段。是指导 Claude 如何创建记忆这一更大工作的一部分。
- [系统提示词：执行任务（宏大任务）](./system-prompts/system-prompt-doing-tasks-ambitious-tasks.md)（**47** tks）- 允许用户完成宏大任务；在范围上遵从用户判断。
- [系统提示词：执行任务（帮助和反馈）](./system-prompts/system-prompt-doing-tasks-help-and-feedback.md)（**24** tks）- 如何告知用户帮助和反馈渠道。
- [系统提示词：执行任务（最小化文件创建）](./system-prompts/system-prompt-doing-tasks-minimize-file-creation.md)（**47** tks）- 优先编辑现有文件而非创建新文件。
- [系统提示词：执行任务（不添加兼容性补丁）](./system-prompts/system-prompt-doing-tasks-no-compatibility-hacks.md)（**52** tks）- 彻底删除未使用的代码，而非添加兼容性垫片。
- [系统提示词：执行任务（不过早抽象）](./system-prompts/system-prompt-doing-tasks-no-premature-abstractions.md)（**72** tks）- 不为一次性操作或假设性需求创建抽象。
- [系统提示词：执行任务（不估计时间）](./system-prompts/system-prompt-doing-tasks-no-time-estimates.md)（**47** tks）- 避免给出时间估计或预测。
- [系统提示词：执行任务（不添加不必要的内容）](./system-prompts/system-prompt-doing-tasks-no-unnecessary-additions.md)（**78** tks）- 不添加未被要求的功能、重构或改进。
- [系统提示词：执行任务（不添加不必要的错误处理）](./system-prompts/system-prompt-doing-tasks-no-unnecessary-error-handling.md)（**64** tks）- 不为不可能发生的场景添加错误处理；仅在边界处进行验证。
- [系统提示词：执行任务（修改前先阅读）](./system-prompts/system-prompt-doing-tasks-read-before-modifying.md)（**46** tks）- 在提出修改建议之前先阅读并理解现有代码。
- [系统提示词：执行任务（安全）](./system-prompts/system-prompt-doing-tasks-security.md)（**67** tks）- 避免引入注入、XSS 等安全漏洞。
- [系统提示词：执行任务（软件工程重点）](./system-prompts/system-prompt-doing-tasks-software-engineering-focus.md)（**104** tks）- 用户主要请求软件工程任务；在该上下文中解释指令。
- [系统提示词：谨慎执行操作](./system-prompts/system-prompt-executing-actions-with-care.md)（**590** tks）- 谨慎执行操作的指令。
- [系统提示词：Fork 使用指南](./system-prompts/system-prompt-fork-usage-guidelines.md)（**419** tks）- 何时 fork 子代理的指令，以及禁止在 fork 运行中途读取 fork 输出或伪造 fork 结果的规则。
- [系统提示词：Git 状态](./system-prompts/system-prompt-git-status.md)（**37** tks）- 在会话开始时显示当前 git 状态的系统提示词。
- [系统提示词：钩子配置](./system-prompts/system-prompt-hooks-configuration.md)（**1493** tks）- 钩子配置的系统提示词。用于上述 Claude Code 配置技能。
- [系统提示词：如何使用 SendUserMessage 工具](./system-prompts/system-prompt-how-to-use-the-sendusermessage-tool.md)（**283** tks）- 使用 SendUserMessage 工具的指令。
- [系统提示词：一览摘要洞察](./system-prompts/system-prompt-insights-at-a-glance-summary.md)（**569** tks）- 为洞察报告生成简洁的四部分摘要（运行良好的方面、阻碍因素、快速改进点、宏大工作流）。
- [系统提示词：洞察摩擦分析](./system-prompts/system-prompt-insights-friction-analysis.md)（**139** tks）- 分析聚合使用数据以识别摩擦模式并对重复出现的问题进行分类。
- [系统提示词：洞察地平线](./system-prompts/system-prompt-insights-on-the-horizon.md)（**148** tks）- 识别宏大的未来工作流和自主 AI 辅助开发的机会。
- [系统提示词：洞察会话维度提取](./system-prompts/system-prompt-insights-session-facets-extraction.md)（**310** tks）- 从单个 Claude Code 会话记录中提取结构化维度（目标类别、满意度、摩擦）。
- [系统提示词：洞察建议](./system-prompts/system-prompt-insights-suggestions.md)（**748** tks）- 生成可操作的建议，包括 CLAUDE.md 补充内容、可尝试的功能和使用模式。
- [系统提示词：学习模式（洞察）](./system-prompts/system-prompt-learning-mode-insights.md)（**142** tks）- 学习模式激活时提供教育性洞察的指令。
- [系统提示词：学习模式](./system-prompts/system-prompt-learning-mode.md)（**1042** tks）- 包含人机协作指令的学习模式主系统提示词。
- [系统提示词：MCP 工具结果截断](./system-prompts/system-prompt-mcp-tool-result-truncation.md)（**164** tks）- 处理 MCP 工具长输出的指南，包括何时使用直接文件查询与子代理进行分析。
- [系统提示词：用户反馈的记忆描述](./system-prompts/system-prompt-memory-description-of-user-feedback.md)（**139** tks）- 描述存储工作方式指导的用户反馈记忆类型，强调记录成功和失败，并检查与团队记忆的矛盾。
- [系统提示词：最小模式](./system-prompts/system-prompt-minimal-mode.md)（**164** tks）- 描述最小模式的行为和约束，该模式跳过钩子、LSP、插件、自动记忆和其他功能，同时需要通过 CLI 标志显式提供上下文。
- [系统提示词：使用 sleep 命令的六条规则之一](./system-prompts/system-prompt-one-of-six-rules-for-using-sleep-command.md)（**23** tks）- 使用 sleep 命令的六条规则之一。
- [系统提示词：选项预览器](./system-prompts/system-prompt-option-previewer.md)（**151** tks）- 以并排布局预览 UI 选项的系统提示词。
- [系统提示词：输出效率](./system-prompts/system-prompt-output-efficiency.md)（**177** tks）- 指示 Claude 在文本输出中简洁直接，以回答为先而非推理，并将响应限制在必要信息范围内。
- [系统提示词：并行工具调用说明（"工具使用策略"的一部分）](./system-prompts/system-prompt-parallel-tool-call-note-part-of-tool-usage-policy.md)（**102** tks）- 告知 Claude 使用并行工具调用的系统提示词。
- [系统提示词：部分压缩指令](./system-prompts/system-prompt-partial-compaction-instructions.md)（**725** tks）- 关于用户决定仅压缩部分会话时如何进行压缩的指令，包含结构化摘要格式和分析流程。
- [系统提示词：计划模式第四阶段](./system-prompts/system-prompt-phase-four-of-plan-mode.md)（**142** tks）- 计划模式的第四阶段。
- [系统提示词：PowerShell 5.1 版本](./system-prompts/system-prompt-powershell-edition-for-51.md)（**285** tks）- 提供 Windows PowerShell 5.1 信息的系统提示词。
- [系统提示词：远程计划模式（ultraplan）](./system-prompts/system-prompt-remote-plan-mode-ultraplan.md)（**617** tks）- 在远程规划会话期间注入的系统提醒，指示 Claude 探索代码库、通过 ExitPlanMode 生成包含图表的计划，并在批准后实施拉取请求。
- [系统提示词：远程规划会话](./system-prompts/system-prompt-remote-planning-session.md)（**432** tks）- 配置远程规划会话的系统提醒，指示探索代码库、通过 ExitPlanMode 生成实施计划，并处理计划批准、拒绝或传送回用户本地终端。
- [系统提示词：暂存目录](./system-prompts/system-prompt-scratchpad-directory.md)（**170** tks）- 使用专用暂存目录存放临时文件的指令。
- [系统提示词：技能化当前会话](./system-prompts/system-prompt-skillify-current-session.md)（**1882** tks）- 将当前会话转换为技能的系统提示词。
- [系统提示词：子代理委派示例](./system-prompts/system-prompt-subagent-delegation-examples.md)（**606** tks）- 提供示例交互，展示协调代理应如何将任务委派给子代理、处理等待状态并报告结果。
- [系统提示词：团队成员通信](./system-prompts/system-prompt-teammate-communication.md)（**130** tks）- 群体中团队成员通信的系统提示词。
- [系统提示词：语气和风格（代码引用）](./system-prompts/system-prompt-tone-and-style-code-references.md)（**39** tks）- 引用代码时包含 file_path:line_number 的指令。
- [系统提示词：语气和风格（简洁输出——简短）](./system-prompts/system-prompt-tone-and-style-concise-output-short.md)（**16** tks）- 简短简洁响应的指令。
- [系统提示词：工具执行被拒绝](./system-prompts/system-prompt-tool-execution-denied.md)（**144** tks）- 工具执行被拒绝时的系统提示词。
- [系统提示词：工具使用（创建文件）](./system-prompts/system-prompt-tool-usage-create-files.md)（**30** tks）- 优先使用 Write 工具而非 cat heredoc 或 echo 重定向。
- [系统提示词：工具使用（委派探索）](./system-prompts/system-prompt-tool-usage-delegate-exploration.md)（**95** tks）- 使用 Task 工具进行更广泛的代码库探索和深度研究。
- [系统提示词：工具使用（直接搜索）](./system-prompts/system-prompt-tool-usage-direct-search.md)（**39** tks）- 直接使用 Glob/Grep 进行简单的定向搜索。
- [系统提示词：工具使用（编辑文件）](./system-prompts/system-prompt-tool-usage-edit-files.md)（**26** tks）- 优先使用 Edit 工具而非 sed/awk。
- [系统提示词：工具使用（读取文件）](./system-prompts/system-prompt-tool-usage-read-files.md)（**29** tks）- 优先使用 Read 工具而非 cat/head/tail/sed。
- [系统提示词：工具使用（保留 Bash）](./system-prompts/system-prompt-tool-usage-reserve-bash.md)（**75** tks）- 将 Bash 工具专门保留用于系统命令和终端操作。
- [系统提示词：工具使用（搜索内容）](./system-prompts/system-prompt-tool-usage-search-content.md)（**30** tks）- 优先使用 Grep 工具而非 grep 或 rg。
- [系统提示词：工具使用（搜索文件）](./system-prompts/system-prompt-tool-usage-search-files.md)（**26** tks）- 优先使用 Glob 工具而非 find 或 ls。
- [系统提示词：工具使用（技能调用）](./system-prompts/system-prompt-tool-usage-skill-invocation.md)（**102** tks）- 斜杠命令通过 Skill 工具调用用户可调用的技能。
- [系统提示词：工具使用（子代理指导）](./system-prompts/system-prompt-tool-usage-subagent-guidance.md)（**103** tks）- 关于何时以及如何有效使用子代理的指导。
- [系统提示词：工具使用（任务管理）](./system-prompts/system-prompt-tool-usage-task-management.md)（**70** tks）- 使用 TodoWrite 分解和跟踪工作进度。
- [系统提示词：工作进程指令](./system-prompts/system-prompt-worker-instructions.md)（**272** tks）- 工作进程在实施变更时遵循的指令。
- [系统提示词：编写子代理提示词](./system-prompts/system-prompt-writing-subagent-prompts.md)（**287** tks）- 向子代理委派任务时编写有效提示词的指南，涵盖继承上下文与全新子代理场景。

### 系统提醒

大型系统提醒的文本内容。

- [系统提醒：/btw 附带问题](./system-prompts/system-reminder-btw-side-question.md)（**244** tks）- 不使用工具的 /btw 斜杠命令附带问题的系统提醒。
- [系统提醒：代理提及](./system-prompts/system-reminder-agent-mention.md)（**45** tks）- 通知用户想要调用某个代理。
- [系统提醒：压缩文件引用](./system-prompts/system-reminder-compact-file-reference.md)（**57** tks）- 会话摘要前读取的文件引用。
- [系统提醒：已退出计划模式](./system-prompts/system-reminder-exited-plan-mode.md)（**73** tks）- 退出计划模式时的通知。
- [系统提醒：文件存在但为空](./system-prompts/system-reminder-file-exists-but-empty.md)（**27** tks）- 读取空文件时的警告。
- [系统提醒：文件被用户或代码检查器修改](./system-prompts/system-reminder-file-modified-by-user-or-linter.md)（**97** tks）- 文件被外部修改的通知。
- [系统提醒：文件在 IDE 中打开](./system-prompts/system-reminder-file-opened-in-ide.md)（**37** tks）- 用户在 IDE 中打开文件的通知。
- [系统提醒：文件短于偏移量](./system-prompts/system-reminder-file-shorter-than-offset.md)（**59** tks）- 文件读取偏移量超过文件长度时的警告。
- [系统提醒：文件已截断](./system-prompts/system-reminder-file-truncated.md)（**74** tks）- 文件因大小而被截断的通知。
- [系统提醒：钩子附加上下文](./system-prompts/system-reminder-hook-additional-context.md)（**35** tks）- 来自钩子的附加上下文。
- [系统提醒：钩子阻止错误](./system-prompts/system-reminder-hook-blocking-error.md)（**52** tks）- 来自阻止性钩子命令的错误。
- [系统提醒：钩子停止续接前缀](./system-prompts/system-reminder-hook-stopped-continuation-prefix.md)（**12** tks）- 钩子停止续接消息的前缀。
- [系统提醒：钩子停止续接](./system-prompts/system-reminder-hook-stopped-continuation.md)（**30** tks）- 钩子停止续接时的消息。
- [系统提醒：钩子成功](./system-prompts/system-reminder-hook-success.md)（**29** tks）- 来自钩子的成功消息。
- [系统提醒：已调用技能](./system-prompts/system-reminder-invoked-skills.md)（**33** tks）- 本次会话中调用的技能列表。
- [系统提醒：在 IDE 中选中的行](./system-prompts/system-reminder-lines-selected-in-ide.md)（**66** tks）- 关于用户在 IDE 中选中的行的通知。
- [系统提醒：MCP 资源无内容](./system-prompts/system-reminder-mcp-resource-no-content.md)（**41** tks）- MCP 资源没有内容时显示。
- [系统提醒：MCP 资源无可显示内容](./system-prompts/system-reminder-mcp-resource-no-displayable-content.md)（**43** tks）- MCP 资源没有可显示内容时显示。
- [系统提醒：Read 工具调用后的恶意软件分析](./system-prompts/system-reminder-malware-analysis-after-read-tool-call.md)（**87** tks）- 分析恶意软件但不改进或增强它的指令。
- [系统提醒：记忆文件内容](./system-prompts/system-reminder-memory-file-contents.md)（**36** tks）- 按路径显示记忆文件的内容。
- [系统提醒：嵌套记忆内容](./system-prompts/system-reminder-nested-memory-contents.md)（**33** tks）- 嵌套记忆文件的内容。
- [系统提醒：检测到新诊断](./system-prompts/system-reminder-new-diagnostics-detected.md)（**35** tks）- 关于新诊断问题的通知。
- [系统提醒：输出样式已激活](./system-prompts/system-reminder-output-style-active.md)（**32** tks）- 输出样式已激活的通知。
- [系统提醒：计划文件引用](./system-prompts/system-reminder-plan-file-reference.md)（**62** tks）- 对现有计划文件的引用。
- [系统提醒：计划模式已激活（5 阶段）](./system-prompts/system-reminder-plan-mode-is-active-5-phase.md)（**1297** tks）- 具有并行探索和多代理规划的增强型计划模式系统提醒。
- [系统提醒：计划模式已激活（迭代式）](./system-prompts/system-reminder-plan-mode-is-active-iterative.md)（**936** tks）- 具有用户访谈工作流的主代理迭代式计划模式系统提醒。
- [系统提醒：计划模式已激活（子代理）](./system-prompts/system-reminder-plan-mode-is-active-subagent.md)（**307** tks）- 子代理的简化计划模式系统提醒。
- [系统提醒：重新进入计划模式](./system-prompts/system-reminder-plan-mode-re-entry.md)（**236** tks）- 用户在通过 shift+tab 或批准 Claude 的计划后退出计划模式，再次进入计划模式时发送的系统提醒。
- [系统提醒：会话续接](./system-prompts/system-reminder-session-continuation.md)（**37** tks）- 会话从另一台机器继续的通知。
- [系统提醒：任务工具提醒](./system-prompts/system-reminder-task-tools-reminder.md)（**123** tks）- 使用任务跟踪工具的提醒。
- [系统提醒：团队协调](./system-prompts/system-reminder-team-coordination.md)（**250** tks）- 团队协调的系统提醒。
- [系统提醒：团队关闭](./system-prompts/system-reminder-team-shutdown.md)（**136** tks）- 团队关闭的系统提醒。
- [系统提醒：TodoWrite 提醒](./system-prompts/system-reminder-todowrite-reminder.md)（**98** tks）- 使用 TodoWrite 工具进行任务跟踪的提醒。
- [系统提醒：词元使用情况](./system-prompts/system-reminder-token-usage.md)（**39** tks）- 当前词元使用统计数据。
- [系统提醒：美元预算](./system-prompts/system-reminder-usd-budget.md)（**42** tks）- 当前美元预算统计数据。
- [系统提醒：Ultraplan 模式](./system-prompts/system-reminder-ultraplan-mode.md)（**437** tks）- 使用 Ultraplan 模式创建包含多代理探索和评审的详细实施计划的系统提醒。
- [系统提醒：验证计划提醒](./system-prompts/system-reminder-verify-plan-reminder.md)（**47** tks）- 验证已完成计划的提醒。

### 内置工具描述

- [工具描述：AskUserQuestion](./system-prompts/tool-description-askuserquestion.md)（**287** tks）- 向用户提问的工具描述。
- [工具描述：Computer](./system-prompts/tool-description-computer.md)（**161** tks）- Chrome 浏览器计算机自动化工具的主要描述。
- [工具描述：Config](./system-prompts/tool-description-config.md)（**275** tks）- 获取和设置 Claude Code 配置的工具，包含使用说明和可配置设置列表。
- [工具描述：CronCreate](./system-prompts/tool-description-croncreate.md)（**948** tks）- 描述用于排队一次性或循环 cron 任务的 CronCreate 工具，包含抖动和非整分钟调度指南。
- [工具描述：Edit](./system-prompts/tool-description-edit.md)（**202** tks）- 在文件中执行精确字符串替换的工具。
- [工具描述：EnterPlanMode](./system-prompts/tool-description-enterplanmode.md)（**878** tks）- 进入计划模式以探索和设计实施方案的工具描述。
- [工具描述：EnterWorktree](./system-prompts/tool-description-enterworktree.md)（**359** tks）- EnterWorktree 工具的工具描述。
- [工具描述：ExitPlanMode](./system-prompts/tool-description-exitplanmode.md)（**417** tks）- ExitPlanMode 工具的描述，该工具向用户呈现计划对话框供其批准。
- [工具描述：ExitWorktree](./system-prompts/tool-description-exitworktree.md)（**527** tks）- 大致上是 ExitWorktree 的逆操作。
- [工具描述：Grep](./system-prompts/tool-description-grep.md)（**300** tks）- 使用 ripgrep 进行内容搜索的工具描述。
- [工具描述：LSP](./system-prompts/tool-description-lsp.md)（**255** tks）- LSP 工具的描述。
- [工具描述：NotebookEdit](./system-prompts/tool-description-notebookedit.md)（**121** tks）- 编辑 Jupyter notebook 单元格的工具描述。
- [工具描述：PowerShell](./system-prompts/tool-description-powershell.md)（**1455** tks）- 描述 PowerShell 命令执行工具，包含语法指南、超时设置，以及对文件操作优先使用专用工具而非 PowerShell 的指令。
- [工具描述：ReadFile](./system-prompts/tool-description-readfile.md)（**473** tks）- 读取文件的工具描述。
- [工具描述：SendMessageTool](./system-prompts/tool-description-sendmessagetool.md)（**362** tks）- 代理团队版本的 SendMessageTool。
- [工具描述：Skill](./system-prompts/tool-description-skill.md)（**326** tks）- 在主会话中执行技能的工具描述。
- [工具描述：TaskCreate](./system-prompts/tool-description-taskcreate.md)（**499** tks）- TaskCreate 工具的工具描述。
- [工具描述：TeamDelete](./system-prompts/tool-description-teamdelete.md)（**154** tks）- TeamDelete 工具的工具描述。
- [工具描述：TeammateTool](./system-prompts/tool-description-teammatetool.md)（**1585** tks）- 在群体中管理团队和协调团队成员的工具。
- [工具描述：TodoWrite](./system-prompts/tool-description-todowrite.md)（**2037** tks）- 创建和管理任务列表的工具描述。
- [工具描述：WebFetch](./system-prompts/tool-description-webfetch.md)（**297** tks）- 网页获取功能的工具描述。
- [工具描述：WebSearch](./system-prompts/tool-description-websearch.md)（**321** tks）- 网页搜索功能的工具描述。
- [工具描述：Write](./system-prompts/tool-description-write.md)（**138** tks）- 向本地文件系统写入文件的工具。

**部分工具描述的附加说明**

- [工具描述：Agent（使用说明）](./system-prompts/tool-description-agent-usage-notes.md)（**798** tks）- Task/Agent 工具的使用说明和指令，包括启动子代理、后台执行、恢复和工作树隔离的指南。
- [工具描述：Agent（何时启动子代理）](./system-prompts/tool-description-agent-when-to-launch-subagents.md)（**186** tks）- 描述_何时_使用 Agent 工具——用于启动专用子代理子进程以自主处理复杂的多步骤任务。
- [工具描述：AskUserQuestion（预览字段）](./system-prompts/tool-description-askuserquestion-preview-field.md)（**134** tks）- 在单选问题选项上使用 HTML 预览字段显示视觉内容（如 UI 模型、代码片段和图表）的指令。
- [工具描述：Bash（git 提交和 PR 创建指令）](./system-prompts/tool-description-bash-git-commit-and-pr-creation-instructions.md)（**1611** tks）- 创建 git 提交和 GitHub 拉取请求（PR）的指令。
- [工具描述：Bash（替代方案——通信）](./system-prompts/tool-description-bash-alternative-communication.md)（**18** tks）- Bash 工具替代方案：直接输出文本而非使用 echo/printf。
- [工具描述：Bash（替代方案——内容搜索）](./system-prompts/tool-description-bash-alternative-content-search.md)（**27** tks）- Bash 工具替代方案：使用 Grep 进行内容搜索而非 grep/rg。
- [工具描述：Bash（替代方案——编辑文件）](./system-prompts/tool-description-bash-alternative-edit-files.md)（**27** tks）- Bash 工具替代方案：使用 Edit 进行文件编辑而非 sed/awk。
- [工具描述：Bash（替代方案——文件搜索）](./system-prompts/tool-description-bash-alternative-file-search.md)（**26** tks）- Bash 工具替代方案：使用 Glob 进行文件搜索而非 find/ls。
- [工具描述：Bash（替代方案——读取文件）](./system-prompts/tool-description-bash-alternative-read-files.md)（**27** tks）- Bash 工具替代方案：使用 Read 进行文件读取而非 cat/head/tail。
- [工具描述：Bash（替代方案——写入文件）](./system-prompts/tool-description-bash-alternative-write-files.md)（**29** tks）- Bash 工具替代方案：使用 Write 进行文件写入而非 echo/cat。
- [工具描述：Bash（内置工具说明）](./system-prompts/tool-description-bash-built-in-tools-note.md)（**53** tks）- 说明内置工具比 Bash 等效工具提供更好的用户体验。
- [工具描述：Bash（git——避免破坏性操作）](./system-prompts/tool-description-bash-git-avoid-destructive-ops.md)（**58** tks）- Bash 工具 git 指令：考虑破坏性操作的更安全替代方案。
- [工具描述：Bash（git——永不跳过钩子）](./system-prompts/tool-description-bash-git-never-skip-hooks.md)（**59** tks）- Bash 工具 git 指令：除非用户要求，否则永不跳过钩子或绕过签名。
- [工具描述：Bash（git——优先新建提交）](./system-prompts/tool-description-bash-git-prefer-new-commits.md)（**22** tks）- Bash 工具 git 指令：优先新建提交而非修改现有提交。
- [工具描述：Bash（维持工作目录）](./system-prompts/tool-description-bash-maintain-cwd.md)（**41** tks）- Bash 工具指令：使用绝对路径并避免使用 cd。
- [工具描述：Bash（不使用换行符）](./system-prompts/tool-description-bash-no-newlines.md)（**24** tks）- Bash 工具指令：不使用换行符分隔命令。
- [工具描述：Bash（概述）](./system-prompts/tool-description-bash-overview.md)（**19** tks）- Bash 工具描述的开头行。
- [工具描述：Bash（并行命令）](./system-prompts/tool-description-bash-parallel-commands.md)（**72** tks）- Bash 工具指令：将独立命令作为并行工具调用运行。
- [工具描述：Bash（优先使用专用工具）](./system-prompts/tool-description-bash-prefer-dedicated-tools.md)（**71** tks）- 警告：对于 find、grep、cat 等操作优先使用专用工具而非 Bash。
- [工具描述：Bash（引用文件路径）](./system-prompts/tool-description-bash-quote-file-paths.md)（**35** tks）- Bash 工具指令：对包含空格的文件路径加引号。
- [工具描述：Bash（沙箱——调整设置）](./system-prompts/tool-description-bash-sandbox-adjust-settings.md)（**26** tks）- 失败时与用户合作调整沙箱设置。
- [工具描述：Bash（沙箱——默认使用沙箱）](./system-prompts/tool-description-bash-sandbox-default-to-sandbox.md)（**38** tks）- 默认使用沙箱；仅在用户要求或有沙箱限制证据时绕过。
- [工具描述：Bash（沙箱——证据列表标题）](./system-prompts/tool-description-bash-sandbox-evidence-list-header.md)（**15** tks）- 沙箱导致失败证据列表的标题。
- [工具描述：Bash（沙箱——证据：访问被拒绝）](./system-prompts/tool-description-bash-sandbox-evidence-access-denied.md)（**15** tks）- 沙箱证据：访问允许目录以外路径被拒绝。
- [工具描述：Bash（沙箱——证据：网络故障）](./system-prompts/tool-description-bash-sandbox-evidence-network-failures.md)（**17** tks）- 沙箱证据：连接到非白名单主机的网络连接失败。
- [工具描述：Bash（沙箱——证据：操作不被允许）](./system-prompts/tool-description-bash-sandbox-evidence-operation-not-permitted.md)（**18** tks）- 沙箱证据：操作不被允许的错误。
- [工具描述：Bash（沙箱——证据：unix 套接字错误）](./system-prompts/tool-description-bash-sandbox-evidence-unix-socket-errors.md)（**11** tks）- 沙箱证据：unix 套接字连接错误。
- [工具描述：Bash（沙箱——说明限制）](./system-prompts/tool-description-bash-sandbox-explain-restriction.md)（**36** tks）- 说明哪项沙箱限制导致了失败。
- [工具描述：Bash（沙箱——失败证据条件）](./system-prompts/tool-description-bash-sandbox-failure-evidence-condition.md)（**48** tks）- 条件：命令失败并有沙箱限制证据。
- [工具描述：Bash（沙箱——强制模式）](./system-prompts/tool-description-bash-sandbox-mandatory-mode.md)（**34** tks）- 策略：所有命令必须在沙箱模式下运行。
- [工具描述：Bash（沙箱——无例外）](./system-prompts/tool-description-bash-sandbox-no-exceptions.md)（**17** tks）- 命令在任何情况下都不能在沙箱外运行。
- [工具描述：Bash（沙箱——不建议敏感路径）](./system-prompts/tool-description-bash-sandbox-no-sensitive-paths.md)（**36** tks）- 不建议将敏感路径添加到沙箱允许列表。
- [工具描述：Bash（沙箱——逐命令处理）](./system-prompts/tool-description-bash-sandbox-per-command.md)（**52** tks）- 逐个处理每个命令；对未来命令默认使用沙箱。
- [工具描述：Bash（沙箱——响应标题）](./system-prompts/tool-description-bash-sandbox-response-header.md)（**17** tks）- 遇到沙箱导致的失败时如何响应的标题。
- [工具描述：Bash（沙箱——不使用沙箱重试）](./system-prompts/tool-description-bash-sandbox-retry-without-sandbox.md)（**33** tks）- 沙箱失败时立即使用 dangerouslyDisableSandbox 重试。
- [工具描述：Bash（沙箱——临时目录）](./system-prompts/tool-description-bash-sandbox-tmpdir.md)（**58** tks）- 在沙箱模式下使用 $TMPDIR 存放临时文件。
- [工具描述：Bash（沙箱——用户权限提示）](./system-prompts/tool-description-bash-sandbox-user-permission-prompt.md)（**14** tks）- 说明禁用沙箱将提示用户授权。
- [工具描述：Bash（分号用法）](./system-prompts/tool-description-bash-semicolon-usage.md)（**29** tks）- Bash 工具指令：当顺序重要但失败无关紧要时使用分号。
- [工具描述：Bash（顺序命令）](./system-prompts/tool-description-bash-sequential-commands.md)（**42** tks）- Bash 工具指令：用 && 链接依赖命令。
- [工具描述：Bash（sleep——保持短暂）](./system-prompts/tool-description-bash-sleep-keep-short.md)（**29** tks）- Bash 工具指令：将 sleep 时长保持在 1-5 秒。
- [工具描述：Bash（sleep——不轮询后台任务）](./system-prompts/tool-description-bash-sleep-no-polling-background-tasks.md)（**37** tks）- Bash 工具指令：不轮询后台任务，等待通知。
- [工具描述：Bash（sleep——立即运行）](./system-prompts/tool-description-bash-sleep-run-immediately.md)（**21** tks）- Bash 工具指令：不在可以立即运行的命令之间使用 sleep。
- [工具描述：Bash（sleep——使用检查命令）](./system-prompts/tool-description-bash-sleep-use-check-commands.md)（**34** tks）- Bash 工具指令：轮询时使用检查命令而非 sleep。
- [工具描述：Bash（超时）](./system-prompts/tool-description-bash-timeout.md)（**83** tks）- Bash 工具指令：可选的超时配置。
- [工具描述：Bash（验证父目录）](./system-prompts/tool-description-bash-verify-parent-directory.md)（**38** tks）- Bash 工具指令：创建文件前验证父目录。
- [工具描述：Bash（工作目录）](./system-prompts/tool-description-bash-working-directory.md)（**37** tks）- Bash 工具关于工作目录持久性和 shell 状态的说明。
- [工具描述：SendMessageTool（非代理团队）](./system-prompts/tool-description-sendmessagetool-non-agent-teams.md)（**133** tks）- 发送用户将读取的消息，对该工具有很好的描述。
- [工具描述：TaskList（团队成员工作流）](./system-prompts/tool-description-tasklist-teammate-workflow.md)（**133** tks）- 附加到 TaskList 工具描述的条件性部分。
- [工具描述：ToolSearch（第二部分）](./system-prompts/tool-description-toolsearch-second-part.md)（**202** tks）- 工具描述的主体部分。
- [工具描述：request_teach_access（教学模式的一部分）](./system-prompts/tool-description-request_teach_access-part-of-teach-mode.md)（**139** tks）- 描述一个请求权限的工具，该工具使用全屏工具提示覆盖层逐步引导用户完成任务，而非直接访问。
- [工具参数：Computer 操作](./system-prompts/tool-parameter-computer-action.md)（**251** tks）- Chrome 浏览器计算机工具的操作参数选项。

### 技能

用于专项任务的内置技能提示词。

- [技能：/init CLAUDE.md 和技能设置（新版本）](./system-prompts/skill-init-claudemd-and-skill-setup-new-version.md)（**4618** tks）- 在当前仓库中设置 CLAUDE.md 及相关技能/钩子的全面引导流程，包括代码库探索、用户访谈和迭代式提案优化。
- [技能：/loop 斜杠命令](./system-prompts/skill-loop-slash-command.md)（**1040** tks）- 将用户输入解析为间隔和提示词，将间隔转换为 cron 表达式，并安排循环任务。
- [技能：/stuck 斜杠命令](./system-prompts/skill-stuck-slash-command.md)（**964** tks）- 诊断冻结或缓慢的 Claude Code 会话。
- [技能：代理设计模式](./system-prompts/skill-agent-design-patterns.md)（**1974** tks）- 在 Claude API 上构建代理的决策启发式参考指南，包括工具界面设计、上下文管理、缓存策略和工具调用组合。
- [技能：使用 Claude API 构建（参考指南）](./system-prompts/skill-build-with-claude-api-reference-guide.md)（**499** tks）- 呈现特定语言参考文档并快速导航任务的模板。
- [技能：使用 Claude API 构建](./system-prompts/skill-build-with-claude-api.md)（**5541** tks）- 使用 Claude 构建 LLM 驱动应用程序的主要路由指南，包括语言检测、界面选择和架构概述。
- [技能：Computer Use MCP](./system-prompts/skill-computer-use-mcp.md)（**1206** tks）- 使用 computer-use MCP 工具的指令，包括工具选择层级、应用访问层级、链接安全和金融操作限制。
- [技能：创建验证器技能](./system-prompts/skill-create-verifier-skills.md)（**2625** tks）- 为 Verify 代理创建验证器技能以自动验证代码变更的提示词。
- [技能：调试](./system-prompts/skill-debugging.md)（**412** tks）- 调试用户在 Claude Code 会话中遇到的问题的指令。
- [技能：简化](./system-prompts/skill-simplify.md)（**877** tks）- 简化代码的指令。
- [技能：更新 Claude Code 配置](./system-prompts/skill-update-claude-code-config.md)（**1255** tks）- 修改 Claude Code 配置文件（settings.json）的技能。
- [技能：验证 CLI 变更（Verify 技能示例）](./system-prompts/skill-verify-cli-changes-example-for-verify-skill.md)（**565** tks）- 作为 Verify 技能一部分的 CLI 变更验证示例工作流。
- [技能：验证服务器/API 变更（Verify 技能示例）](./system-prompts/skill-verify-serverapi-changes-example-for-verify-skill.md)（**612** tks）- 作为 Verify 技能一部分的服务器/API 变更验证示例工作流。
- [技能：Verify 技能](./system-prompts/skill-verify-skill.md)（**2158** tks）- 用于验证代码变更的主观验证工作流技能。
- [技能：update-config（7 步验证流程）](./system-prompts/skill-update-config-7-step-verification-flow.md)（**1160** tks）- 引导 Claude 完成 7 步流程以构建和验证 Claude Code 钩子的技能，确保它们在用户特定的项目环境中正常工作。
