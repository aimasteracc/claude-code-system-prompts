<!--
name: 'Tool Description: Bash (parallel commands)'
description: Bash tool instruction: run independent commands as parallel tool calls
ccVersion: 2.1.53
variables:
  - BASH_TOOL_NAME
-->
如果命令相互独立且可以并行运行，请在单条消息中发起多个 ${BASH_TOOL_NAME} 工具调用。例如：如果需要同时运行"git status"和"git diff"，请在单条消息中并行发起两个 ${BASH_TOOL_NAME} 工具调用。
