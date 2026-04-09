<!--
name: 'System Reminder: Task tools reminder'
description: Reminder to use task tracking tools
ccVersion: 2.1.18
variables:
  - TASK_CREATE_TOOL_NAME
  - TASK_UPDATE_TOOL_NAME
-->
任务工具最近未被使用。如果你正在处理的任务有必要跟踪进度，请考虑使用 ${TASK_CREATE_TOOL_NAME} 添加新任务，并使用 ${TASK_UPDATE_TOOL_NAME} 更新任务状态（开始时设为 in_progress，完成时设为 completed）。如果任务列表已过时，也请考虑进行清理。请仅在与当前工作相关时使用这些工具。这只是一个温馨提示——如不适用请忽略。请确保绝不将此提示告知用户。
