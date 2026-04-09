<!--
name: 'Agent Prompt: Dream memory pruning'
description: Instructs an agent to perform a memory pruning pass by deleting stale or invalidated memory files and collapsing duplicates in the memory directory
ccVersion: 2.1.94
variables:
  - MEMORY_DIR
  - MEMORY_DIR_CONTEXT
  - ADDITIONAL_CONTEXT
-->
# Dream：记忆剪枝

你正在执行一次 dream——对记忆文件进行剪枝遍历。任务很小：删除过时或已失效的记忆，并合并重复项。

记忆目录：`${MEMORY_DIR}`
${MEMORY_DIR_CONTEXT}

记忆文件是不可变的：绝不要就地编辑。合并意味着删除旧文件，并在需要时写入一个新的单事实文件作为替换。

## 操作步骤

1. 执行 `find ${MEMORY_DIR} -name '*.md'` 以枚举每个记忆文件（包括任何 `team/` 子目录）。
2. 对每个记忆文件，做出判断：
   - **过时或已失效** — 该事实不再成立（与当前代码相矛盾、项目已推进、用户偏好已改变）。删除该文件。
   - **重复或近似重复** — 另一条记忆已涵盖相同事实。删除冗余副本。如果一条更丰富的单事实记忆能替代整个集群，则删除该集群并写入一个新文件（使用系统提示词中自动记忆部分的格式和类型约定）。写入合并替换文件时，从最早源记忆的前置信息（frontmatter）中复制 `created:` 日期，以保持清单排序的准确性。
   - **仍然有效** — 保持不变。

返回关于删除、合并或保留内容的简要摘要。如果没有任何变化，请如实说明。${ADDITIONAL_CONTEXT?`

## 附加上下文

${ADDITIONAL_CONTEXT}`:""}
