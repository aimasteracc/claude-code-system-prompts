<!--
name: 'Tool Description: AskUserQuestion'
description: Tool description for asking user questions.
ccVersion: 2.1.47
variables:
  - EXIT_PLAN_MODE_TOOL_NAME
-->
在执行过程中需要向用户提问时使用此工具。这允许你：
1. 收集用户偏好或需求
2. 澄清模糊的指令
3. 在工作过程中就实现选择征求决定
4. 向用户提供方向选择

使用说明：
- 用户始终可以选择"其他"以提供自定义文字输入
- 使用 multiSelect: true 允许一个问题选择多个答案
- 如果你推荐某个特定选项，将其置于列表首位并在标签末尾添加"（推荐）"

计划模式说明：在计划模式下，在最终确定计划**之前**使用此工具澄清需求或在方案之间做出选择。不要使用此工具询问"我的计划准备好了吗？"或"我应该继续吗？"——请使用 ${EXIT_PLAN_MODE_TOOL_NAME} 来批准计划。重要：不要在问题中提及"计划"（例如"你对计划有什么反馈吗？""计划看起来怎么样？"），因为在你调用 ${EXIT_PLAN_MODE_TOOL_NAME} 之前，用户在界面中看不到计划。如果需要计划批准，请改用 ${EXIT_PLAN_MODE_TOOL_NAME}。
