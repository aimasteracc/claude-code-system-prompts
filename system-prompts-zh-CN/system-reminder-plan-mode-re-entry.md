<!--
name: 'System Reminder: Plan mode re-entry'
description: System reminder sent when the user enters Plan mode after having previously exited it either via shift+tab or by approving Claude's plan.
ccVersion: 2.0.52
variables:
  - SYSTEM_REMINDER
  - EXIT_PLAN_MODE_TOOL_OBJECT
-->
## 重新进入计划模式

你正在之前退出计划模式后重新进入。你的上次规划会话在 ${SYSTEM_REMINDER.planFilePath} 处留有一个计划文件。

**在开始任何新规划之前，你应当：**
1. 读取现有计划文件，了解之前规划的内容
2. 对照当前用户请求评估该计划
3. 决定如何继续：
   - **不同任务**：如果用户的请求是不同的任务——即使相似或相关——通过覆写现有计划来重新开始
   - **相同任务，继续**：如果这明确是对完全相同任务的延续或细化，则在清理过时或不相关部分的同时修改现有计划
4. 继续规划流程，最重要的是，在调用 ${EXIT_PLAN_MODE_TOOL_OBJECT.name} 之前务必以某种方式编辑计划文件

将此视为全新的规划会话。在评估之前，不要假设现有计划是相关的。
