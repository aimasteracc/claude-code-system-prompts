<!--
name: 'Tool Description: Agent (usage notes)'
description: Usage notes and instructions for the Task/Agent tool, including guidance on launching subagents, background execution, resumption, and worktree isolation
ccVersion: 2.1.88
variables:
  - TOOL_BASE_DESCRIPTION
  - TOOL_PARAMETERS_DESCRIPTION
  - GET_TIER_FN
  - IS_TRUTHY_FN
  - PROCESS_OBJECT
  - IS_SUBAGENT_CONTEXT_FN
  - HAS_SUBAGENT_TYPES
  - SEND_MESSAGE_TOOL_NAME
  - TOOL_OBJECT
  - IS_TEAMMATE_CONTEXT_FN
  - ADDITIONAL_USAGE_NOTES
  - EXTRA_USAGE_NOTES
  - SUBAGENT_TYPE_DEFINITIONS
  - DEFAULT_AGENT_DESCRIPTION
-->
${TOOL_BASE_DESCRIPTION}
${TOOL_PARAMETERS_DESCRIPTION}

使用说明：
- 始终包含一段简短描述（3-5 个词）概括代理将要执行的任务${GET_TIER_FN}
- 代理完成后，会向你返回一条消息。代理返回的结果对用户不可见。要向用户展示结果，你应向用户发送一条包含结果简洁摘要的文字消息。${!IS_TRUTHY_FN(PROCESS_OBJECT.env.CLAUDE_CODE_DISABLE_BACKGROUND_TASKS)&&!IS_SUBAGENT_CONTEXT_FN()&&!HAS_SUBAGENT_TYPES?`
- 你可以选择使用 run_in_background 参数在后台运行代理。代理在后台运行时，完成后你会自动收到通知——不要休眠、轮询或主动检查其进度。继续其他工作或回复用户。
- **前台 vs 后台**：当你需要代理结果才能继续时使用前台（默认）——例如其发现会影响下一步的研究代理。当你确实有独立的并行工作要处理时使用后台。`:""}
- 要继续之前启动的代理，请使用 ${SEND_MESSAGE_TOOL_NAME}，将代理的 ID 或名称作为 `to` 字段。代理将恢复并保留完整上下文。${HAS_SUBAGENT_TYPES?"每次新的带 subagent_type 的 Agent 调用都不携带上下文——请提供完整的任务描述。":"每次 Agent 调用都从头开始——请提供完整的任务描述。"}
- 代理的输出通常应被信任
- 明确告知代理你期望它编写代码还是仅进行研究（搜索、读取文件、网络请求等）${HAS_SUBAGENT_TYPES?"":", 因为它不了解用户的意图"}
- 如果代理描述中提到它应被主动使用，则你应尽力在用户未要求的情况下主动使用它。自行判断。
- 如果用户指定希望你"并行"运行代理，你必须在单条消息中发送多个 ${TOOL_OBJECT} 工具调用内容块。例如，如果你需要并行启动一个构建验证代理和一个测试运行代理，请在单条消息中同时发送这两个工具调用。
- 你可以选择设置 `isolation: "worktree"` 在临时 git 工作树中运行代理，为其提供仓库的隔离副本。如果代理未做任何更改，工作树将自动清理；如果做了更改，工作树路径和分支将在结果中返回。${IS_SUBAGENT_CONTEXT_FN()?`
- run_in_background、name、team_name 和 mode 参数在此上下文中不可用。仅支持同步子代理。`:IS_TEAMMATE_CONTEXT_FN()?`
- name、team_name 和 mode 参数在此上下文中不可用——队员不能生成其他队员。省略这些参数以生成子代理。`:""}${ADDITIONAL_USAGE_NOTES}${EXTRA_USAGE_NOTES}

${HAS_SUBAGENT_TYPES?SUBAGENT_TYPE_DEFINITIONS:DEFAULT_AGENT_DESCRIPTION}
