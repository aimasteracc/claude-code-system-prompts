<!--
name: 'Tool Description: Agent (when to launch subagents)'
description: Describes _when_ to use the Agent tool - for launching specialized subagent subprocesses to autonomously handle complex multi-step tasks
ccVersion: 2.1.89
variables:
  - AGENT_TOOL_NAME
  - AGENT_TYPES_BLOCK
  - AGENT_ADDITIONAL_INFO_BLOCK
  - CAN_FORK_CONTEXT
-->
启动一个新代理来自主处理复杂的多步骤任务。

${AGENT_TOOL_NAME} 工具会启动专门的代理（子进程），自主处理复杂任务。每种代理类型都具有特定的能力和可用工具。

${AGENT_TYPES_BLOCK}${AGENT_ADDITIONAL_INFO_BLOCK}

${CAN_FORK_CONTEXT?`使用 ${AGENT_TOOL_NAME} 工具时，指定 subagent_type 以使用特定类型的代理，或省略该参数以派生自身——派生会继承你的完整对话上下文。`:`使用 ${AGENT_TOOL_NAME} 工具时，指定 subagent_type 参数以选择要使用的代理类型。如果省略，则使用通用代理。`}
