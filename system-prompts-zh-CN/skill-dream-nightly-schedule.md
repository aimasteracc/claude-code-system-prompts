<!--
name: 'Skill: /dream nightly schedule'
description: Sets up a recurring nightly memory consolidation job by deduplicating existing schedules, creating a new cron task, confirming details to the user, and running an immediate consolidation
ccVersion: 2.1.97
variables:
  - CRON_LIST_TOOL_NAME
  - CRON_DELETE_TOOL_NAME
  - CRON_CREATE_TOOL_NAME
  - CRON_EXPRESSION
  - SCHEDULED_TIME_LOCAL
  - CANCEL_TIMEFRAME_DAYS
  - CONSOLIDATE_SKILL_FN
  - CONSOLIDATE_PROMPT
  - MEMORY_STORE_PATH
  - CONSOLIDATION_OPTIONS
-->
# Dream：安排每夜巩固任务

用户希望设置一个每夜重复执行的记忆巩固任务。

**第 1 步——去重现有夜间任务**

调用 ${CRON_LIST_TOOL_NAME}，检查是否存在提示词为 `"/dream consolidate"` 的任务。如果存在，先用 ${CRON_DELETE_TOOL_NAME} 删除它，以免续期时产生重叠的任务。

**第 2 步——创建计划**

调用 ${CRON_CREATE_TOOL_NAME}，参数如下：
- `cron`: `"${CRON_EXPRESSION}"`
- `prompt`: `"/dream consolidate"`
- `recurring`: true
- `durable`: true

（`consolidate` 后缀确保此提示词在触发时不会匹配 SCHEDULING_KEYWORDS（从而走巩固路径），不会精确匹配 migrateAssistantTasksPermanent() 的 `'/dream'` 检查（从而保持非永久状态），并通过主名称在捆绑技能和磁盘技能上都能正确解析（从而在通过 kill-switch 或 KAIROS 激活禁用捆绑技能时仍能正常工作）。）

**第 3 步——确认**

告知用户：
- /dream 将每夜约在 ${SCHEDULED_TIME_LOCAL} 本地时间运行，以巩固和整理记忆
- 计划跨会话持久存在（写入 .claude/scheduled_tasks.json）
- 重复任务在 ${CANCEL_TIMEFRAME_DAYS} 天后自动过期——重新运行 `/dream nightly` 即可续期
- 随时可通过 ${CRON_DELETE_TOOL_NAME} 取消（需附上任务 ID）

**第 4 步——立即执行一次巩固**

${CONSOLIDATE_SKILL_FN(CONSOLIDATE_PROMPT,MEMORY_STORE_PATH,CONSOLIDATION_OPTIONS)}
