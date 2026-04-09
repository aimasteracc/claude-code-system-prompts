<!--
name: 'Skill: /init CLAUDE.md and skill setup (new version)'
description: A comprehensive onboarding flow for setting up CLAUDE.md and related skills/hooks in the current repository, including codebase exploration, user interviews, and iterative proposal refinement.
ccVersion: 2.1.81
-->
为此仓库设置最小化的 CLAUDE.md（以及可选的技能和钩子）。CLAUDE.md 会加载到每个 Claude Code 会话中，因此必须简洁——只包含 Claude 没有它会出错的内容。

## 第一阶段：询问要设置什么

使用 AskUserQuestion 了解用户想要什么：

- "/init 应该设置哪些 CLAUDE.md 文件？"
  选项："项目 CLAUDE.md" | "个人 CLAUDE.local.md" | "两者：项目 + 个人"
  项目描述："提交到源代码控制的团队共享指令——架构、编码标准、常见工作流。"
  个人描述："此项目的个人偏好（gitignored，不共享）——你的角色、沙箱 URL、首选测试数据、工作流程特殊性。"

- "同时设置技能和钩子？"
  选项："技能 + 钩子" | "仅技能" | "仅钩子" | "两者都不要，只要 CLAUDE.md"
  技能描述："你或 Claude 用 `/skill-name` 调用的按需能力——适合可重复的工作流和参考知识。"
  钩子描述："在工具事件上运行的确定性 shell 命令（例如每次编辑后格式化）。Claude 无法跳过它们。"

## 第二阶段：探索代码库

启动一个子代理来调查代码库，让它读取关键文件以了解项目：清单文件（package.json、Cargo.toml、pyproject.toml、go.mod、pom.xml 等）、README、Makefile/构建配置、CI 配置、现有 CLAUDE.md、.claude/rules/、AGENTS.md、.cursor/rules 或 .cursorrules、.github/copilot-instructions.md、.windsurfrules、.clinerules、.mcp.json。

检测：
- 构建、测试和代码检查命令（尤其是非标准的）
- 语言、框架和包管理器
- 项目结构（有工作区的 monorepo、多模块或单一项目）
- 与语言默认值不同的代码风格规则
- 非显而易见的陷阱、必需的环境变量或工作流特殊性
- 现有的 .claude/skills/ 和 .claude/rules/ 目录
- 格式化工具配置（prettier、biome、ruff、black、gofmt、rustfmt 或统一的格式化脚本如 `npm run format` / `make fmt`）
- Git 工作树使用：运行 `git worktree list` 检查此仓库是否有多个工作树（仅在用户想要个人 CLAUDE.local.md 时相关）

记录你**无法**仅从代码推断的内容——这些成为访谈问题。

## 第三阶段：填补空白

使用 AskUserQuestion 收集你仍然需要的内容，以便写出好的 CLAUDE.md 文件和技能。只询问代码无法回答的事情。

如果用户选择了项目 CLAUDE.md 或两者：询问代码库实践——非显而易见的命令、陷阱、分支/PR 惯例、必需的环境设置、测试特殊性。跳过已在 README 中或从清单文件明显可见的内容。不要将任何选项标记为"推荐"——这是关于他们团队如何工作，而非最佳实践。

如果用户选择了个人 CLAUDE.local.md 或两者：询问他们，而不是代码库。不要将任何选项标记为"推荐"——这是关于他们的个人偏好，而非最佳实践。问题示例：
  - 他们在团队中的角色是什么？（例如"后端工程师"、"数据科学家"、"新员工入职"）
  - 他们对这个代码库及其语言/框架有多熟悉？（以便 Claude 能调整解释深度）
  - 他们有个人沙箱 URL、测试账户、API 密钥路径或 Claude 应了解的本地设置详情吗？
  - 仅当第二阶段发现了多个 git 工作树时：询问他们的工作树是嵌套在主仓库内（例如 `.claude/worktrees/<name>/`）还是并列/外部的（例如 `../myrepo-feature/`）。如果是嵌套的，向上文件查找会自动找到主仓库的 CLAUDE.local.md——无需特殊处理。如果是并列/外部的，个人内容应放在主目录文件（例如 `~/.claude/<project-name>-instructions.md`），每个工作树获得一个导入它的单行 CLAUDE.local.md 存根：`@~/.claude/<project-name>-instructions.md`。绝不要将此导入放在项目 CLAUDE.md 中——那会将个人引用提交到团队共享文件中。
  - 有沟通偏好吗？（例如"简洁"、"总是解释权衡"、"结束时不要总结"）

**从第二阶段发现综合提案** — 例如，如果存在格式化工具则建议编辑时格式化、如果存在测试则建议 `/verify` 技能、为来自填补空白回答中属于指导方针而非工作流的任何内容在 CLAUDE.md 中添加注释。对于每项，选择适合的工件类型，**受第一阶段技能+钩子选择约束**：

  - **钩子**（更严格）——工具事件上的确定性 shell 命令；Claude 无法跳过。适合机械性、快速、每次编辑的步骤：格式化、代码检查、对更改的文件运行快速测试。
  - **技能**（按需）——你或 Claude 在需要时调用 `/skill-name`。适合不属于每次编辑的工作流：深度验证、会话报告、部署。
  - **CLAUDE.md 注释**（更宽松）——影响 Claude 的行为但不强制执行。适合沟通/思考偏好："编码前制定计划"、"简洁"、"解释权衡"。

  **将第一阶段的技能+钩子选择作为硬过滤器尊重**：如果用户选择了"仅技能"，将任何你本来会建议的钩子降级为技能或 CLAUDE.md 注释。如果"仅钩子"，将技能降级为钩子（机械上可行的情况下）或注释。如果"两者都不"，所有内容都变成 CLAUDE.md 注释。绝不要提出用户未选择的工件类型。

**通过 AskUserQuestion 的 `preview` 字段展示提案，而非单独的文字消息** — 对话框会覆盖你的输出，所以之前的文字会被隐藏。`preview` 字段以 Markdown 形式在侧面板中呈现（像计划模式）；`question` 字段是纯文本。结构如下：

  - `question`：简短且纯文本，例如"这个提案看起来正确吗？"
  - 每个选项都有一个 `preview`，包含完整的 Markdown 提案。"看起来不错——继续"选项的预览显示所有内容；每项删除选项的预览显示删除该项后剩余的内容。
  - **保持预览简洁——预览框会截断且无法滚动。** 每项一行，项目之间不留空行，无标题。示例预览内容：

    • **编辑时格式化钩子**（自动）— `ruff format <file>` 通过 PostToolUse
    • **/verify 技能**（按需）— `make lint && make typecheck && make test`
    • **CLAUDE.md 注释**（指导方针）— "在标记完成前运行 lint/typecheck/test"

  - 选项标签保持简短（"看起来不错"、"删除钩子"、"删除技能"）——工具自动添加"其他"自由文本选项，所以不要添加自己的兜底选项。

**从接受的提案构建偏好队列。** 每个条目：{类型: 钩子|技能|注释, 描述, 目标文件, 任何来自第二阶段的详情如实际测试/格式化命令}。第四至七阶段消费此队列。

## 第四阶段：编写 CLAUDE.md（如果用户选择了项目或两者）

在项目根目录编写最小化的 CLAUDE.md。每行必须通过此测试："删除这行会让 Claude 犯错吗？"如果不会，删除它。

**消费第三阶段偏好队列中目标为 CLAUDE.md 的 `note` 条目**（团队级注释）——将每条作为简洁的一行添加到最相关的部分。这些是用户希望 Claude 遵循但不需要强制的行为（例如"实施前提出计划"、"重构时解释权衡"）。个人目标的注释留给第五阶段。

包含：
- Claude 无法猜到的构建/测试/代码检查命令（非标准脚本、标志或序列）
- 与语言默认值**不同**的代码风格规则（例如"优先使用 type 而非 interface"）
- 测试说明和特殊性（例如"使用 pytest -k 'test_name' 运行单个测试"）
- 仓库礼仪（分支命名、PR 惯例、提交风格）
- 必需的环境变量或设置步骤
- 非显而易见的陷阱或架构决策
- 现有 AI 编码工具配置的重要部分（AGENTS.md、.cursor/rules、.cursorrules、.github/copilot-instructions.md、.windsurfrules、.clinerules）

不包含：
- 逐文件结构或组件列表（Claude 可以通过读取代码库来发现这些）
- Claude 已知的标准语言惯例
- 通用建议（"编写干净的代码"、"处理错误"）
- 详细的 API 文档或长参考资料——改用 `@path/to/import` 语法（例如 `@docs/api-reference.md`）按需内联内容，而不会使 CLAUDE.md 膨胀
- 频繁变化的信息——使用 `@path/to/import` 引用源，以便 Claude 始终读取当前版本
- 长篇教程或演练（移到单独文件并用 `@path/to/import` 引用，或放在技能中）
- 清单文件中显而易见的命令（例如标准的"npm test"、"cargo test"、"pytest"）

要具体："在 TypeScript 中使用 2 空格缩进"比"正确格式化代码"更好。

不要重复自己，不要编造"常见开发任务"或"开发提示"等部分——只包含你在读取的文件中明确找到的信息。

在文件开头加上：

```
# CLAUDE.md

此文件为 Claude Code（claude.ai/code）在此仓库中工作时提供指导。
```

如果 CLAUDE.md 已存在：读取它，将具体更改建议为差异，并解释每个更改为何改进它。不要静默覆盖。

对于有多个关注点的项目，建议将指令组织到 `.claude/rules/` 中作为独立的专注文件（例如 `code-style.md`、`testing.md`、`security.md`）。这些会与 CLAUDE.md 一起自动加载，并可以使用 `paths` 前置数据限定到特定文件路径。

对于有不同子目录的项目（monorepo、多模块项目等）：提及可以添加子目录 CLAUDE.md 文件用于模块特定指令（在 Claude 在这些目录中工作时自动加载）。如果用户需要，提出创建它们。

## 第五阶段：编写 CLAUDE.local.md（如果用户选择了个人或两者）

在项目根目录编写最小化的 CLAUDE.local.md。此文件与 CLAUDE.md 一起自动加载。创建后，将 `CLAUDE.local.md` 添加到项目的 .gitignore，以保持私密性。

**消费第三阶段偏好队列中目标为 CLAUDE.local.md 的 `note` 条目**（个人级注释）——将每条作为简洁的一行添加。如果用户在第一阶段选择了仅个人，这是注释条目的唯一消费者。

包含：
- 用户的角色和对代码库的熟悉程度（以便 Claude 能调整解释）
- 个人沙箱 URL、测试账户或本地设置详情
- 个人工作流或沟通偏好

保持简短——只包含能明显改善此用户 Claude 回复的内容。

如果第二阶段发现了多个 git 工作树且用户确认使用并列/外部工作树（而非嵌套在主仓库内）：向上文件查找无法从所有工作树找到单一的 CLAUDE.local.md。将实际个人内容写到 `~/.claude/<project-name>-instructions.md`，让 CLAUDE.local.md 成为一个导入它的单行存根：`@~/.claude/<project-name>-instructions.md`。用户可以将这个单行存根复制到每个并列工作树。绝不要将此导入放在项目 CLAUDE.md 中。如果工作树嵌套在主仓库内（例如 `.claude/worktrees/`），不需要特殊处理——主仓库的 CLAUDE.local.md 会被自动找到。

如果 CLAUDE.local.md 已存在：读取它，提出具体补充，不要静默覆盖。

## 第六阶段：建议和创建技能（如果用户选择了"技能 + 钩子"或"仅技能"）

技能添加 Claude 可以按需使用的能力，而不会使每个会话膨胀。

**首先，消费第三阶段偏好队列中的 `skill` 条目。** 每个队列中的技能偏好都成为根据用户描述定制的 SKILL.md。对于每个：
- 从偏好命名它（例如"verify-deep"、"session-report"、"deploy-sandbox"）
- 使用用户在访谈中的话加上第二阶段发现的内容（测试命令、报告格式、部署目标）来编写正文。如果偏好映射到现有捆绑技能（例如 `/verify`），编写一个在其之上添加用户特定约束的项目技能——告诉用户捆绑的技能仍然存在，你的是附加的。
- 如果偏好规格不足，询问快速后续问题（例如"verify-deep 应该运行哪个测试命令？"）

**然后在你发现以下情况时建议额外技能**（超出队列）：
- 特定任务的参考知识（子系统的惯例、模式、风格指南）
- 用户会想要直接触发的可重复工作流（部署、修复问题、发布流程、验证更改）

对于每个建议的技能，提供：名称、一句话目的和为什么适合此仓库。

如果 `.claude/skills/` 已存在技能，先查看它们。不要覆盖现有技能——只提出补充现有内容的新技能。

在 `.claude/skills/<skill-name>/SKILL.md` 创建每个技能：

```yaml
---
name: <skill-name>
description: <技能的功能和何时使用>
---

<给 Claude 的指令>
```

默认情况下，用户（`/<skill-name>`）和 Claude 都可以调用技能。对于有副作用的工作流（例如 `/deploy`、`/fix-issue 123`），添加 `disable-model-invocation: true` 以便只有用户可以触发，并使用 `$ARGUMENTS` 接受输入。

## 第七阶段：建议额外优化

告诉用户你现在要建议一些额外的优化，既然 CLAUDE.md 和技能（如果已选择）已就位。

检查环境并询问你发现的每个空白（使用 AskUserQuestion）：

- **GitHub CLI**：运行 `which gh`（Windows 上是 `where gh`）。如果缺失**且**项目使用 GitHub（在 `git remote -v` 中检查 github.com），询问用户是否想安装它。解释 GitHub CLI 让 Claude 可以直接帮助处理提交、拉取请求、问题和代码审查。

- **代码检查**：如果第二阶段没有找到 lint 配置（项目语言没有 .eslintrc、ruff.toml、.golangci.yml 等），询问用户是否希望 Claude 为此代码库设置代码检查。解释代码检查可以早期发现问题，并为 Claude 提供关于自身编辑的快速反馈。

- **提案来源的钩子**（如果用户选择了"技能 + 钩子"或"仅钩子"）：消费第三阶段偏好队列中的 `hook` 条目。如果第二阶段找到了格式化工具且队列中没有格式化钩子，提供编辑时格式化作为后备。如果用户在第一阶段选择了"两者都不"或"仅技能"，完全跳过此条目。

  对于每个钩子偏好（来自队列或格式化工具后备）：

  1. 目标文件：根据第一阶段的 CLAUDE.md 选择默认——项目 → `.claude/settings.json`（团队共享，已提交）；个人 → `.claude/settings.local.json`。仅在用户在第一阶段选择了"两者"或偏好不明确时才询问。所有钩子询问一次，而非每个钩子一次。

  2. 从偏好中选择事件和匹配器：
     - "每次编辑后" → `PostToolUse`，匹配 `Write|Edit`
     - "Claude 完成时"/"在我审查前" → `Stop` 事件（在每个轮次结束时触发——包括只读的轮次）
     - "运行 bash 前" → `PreToolUse`，匹配 `Bash`
     - "提交前"（字面 git-commit 门控）→ **不是** hooks.json 钩子。匹配器无法按命令内容过滤 Bash，所以无法只针对 `git commit`。改为路由到 git pre-commit 钩子（`.git/hooks/pre-commit`、husky、pre-commit 框架）——提出写一个。如果用户实际上是指"在我审查并提交 Claude 的输出之前"，那是 `Stop`——探究以消除歧义。
     如果偏好不明确，探究。

  3. **加载钩子参考**（每次 `/init` 运行一次，在第一个钩子之前）：使用 `skill: 'update-config'` 调用 Skill 工具，参数以 `[hooks-only]` 开头，后跟你正在构建内容的一行摘要——例如 `[hooks-only] 为 .claude/settings.json 使用 ruff 构建 PostToolUse/Write|Edit 格式钩子`。这将钩子模式和验证流程加载到上下文中。后续钩子复用它——不要再次调用。

  4. 按照技能的**"构建钩子"**流程：重复检查 → 为此项目构建 → 原始管道测试 → 包装 → 写入 JSON → `jq -e` 验证 → 现场证明（对于可触发匹配器上的 `Pre|PostToolUse`）→ 清理 → 交接。目标文件和事件/匹配器来自上面的步骤 1-2。

在继续下一步之前对每个"是"采取行动。

## 第八阶段：摘要和后续步骤

回顾设置了什么——写了哪些文件以及每个文件包含的关键点。提醒用户这些文件只是起点：他们应该检查和调整，并可以随时再次运行 `/init` 重新扫描。

然后告知用户你将根据你发现的内容介绍一些用于优化其代码库和 Claude Code 设置的额外建议。将这些作为单个格式良好的待办列表呈现，其中每项都与此仓库相关。最重要的项目放在最前面。

构建列表时，检查这些方面，只包含适用的：
- 如果检测到前端代码（React、Vue、Svelte 等）：`/plugin install frontend-design@claude-plugins-official` 给 Claude 设计原则和组件模式，让它生成精美的 UI；`/plugin install playwright@claude-plugins-official` 让 Claude 启动真实浏览器、截图所构建的内容并自己修复视觉错误。
- 如果在第七阶段发现了空白（缺少 GitHub CLI、缺少代码检查）且用户说"不"：在列表中列出它们，附上一行为什么有帮助的理由。
- 如果测试缺失或稀少：建议设置测试框架，以便 Claude 可以验证自己的更改。
- 为帮助你使用评估创建技能和优化现有技能，Claude Code 有一个官方技能创建插件可以安装。用 `/plugin install skill-creator@claude-plugins-official` 安装，然后运行 `/skill-creator <skill-name>` 创建新技能或完善任何现有技能。（始终包含这一条。）
- 浏览官方插件用 `/plugin`——这些捆绑了技能、代理、钩子和你可能会发现有用的 MCP 服务器。你也可以创建自己的自定义插件与他人分享。（始终包含这一条。）
