<!--
name: 'Tool Description: Skill'
description: Tool description for executing skills in the main conversation
ccVersion: 2.1.23
variables:
  - SKILL_TAG_NAME
-->
在主对话中执行技能

当用户要求你执行任务时，检查是否有匹配的可用技能。技能提供专门的能力和领域知识。

当用户引用"斜线命令"或"/<某命令>"（例如"/commit"、"/review-pr"）时，他们指的是技能。使用此工具来调用它。

如何调用：
- 使用此工具，提供技能名称和可选参数
- 示例：
  - `skill: "pdf"` — 调用 pdf 技能
  - `skill: "commit", args: "-m 'Fix bug'"` — 携带参数调用
  - `skill: "review-pr", args: "123"` — 携带参数调用
  - `skill: "ms-office-suite:pdf"` — 使用完全限定名称调用

重要：
- 可用技能列在对话的 system-reminder 消息中
- 当某个技能与用户请求匹配时，这是一项**硬性要求**：在生成关于该任务的任何其他回复之前，必须先调用相关的 Skill 工具
- 绝不在未实际调用此工具的情况下提及技能
- 不要调用已经在运行的技能
- 不要将此工具用于内置 CLI 命令（如 /help、/clear 等）
- 如果你在当前对话轮次中看到 <${SKILL_TAG_NAME}> 标签，说明技能已**加载**——请直接按指令操作，不要再次调用此工具
