<!--
name: 'Skill: Debugging'
description: Instructions for debugging an issue that the user is encountering in the Claude Code session
ccVersion: 2.1.71
variables:
  - DEBUG_LOGGING_WAS_ALREADY_ACTIVE
  - DEBUG_LOG_PATH
  - DEBUG_LOG_SUMMARY
  - ISSUE_DESCRIPTION
  - GET_SETTINGS_FILE_PATH_FN
  - LOG_LINE_COUNT
  - CLAUDE_CODE_GUIDE_SUBAGENT_NAME
-->
# 调试技能

帮助用户调试在当前 Claude Code 会话中遇到的问题。
${DEBUG_LOGGING_WAS_ALREADY_ACTIVE?"":`
## 调试日志刚已启用

此会话的调试日志在此之前处于**关闭**状态。在此 /debug 调用之前的任何内容均未被捕获。

请告知用户调试日志现在已在 \`${DEBUG_LOG_PATH}\` 激活，请他们重现问题，然后重新读取日志。如果无法重现，他们也可以使用 \`claude --debug\` 重启以从启动就开始捕获日志。
`}
## 会话调试日志

当前会话的调试日志位于：`${DEBUG_LOG_PATH}`

${DEBUG_LOG_SUMMARY}

如需更多上下文，请在整个文件中 grep [ERROR] 和 [WARN] 行。

## 问题描述

${ISSUE_DESCRIPTION||"用户未描述具体问题。请读取调试日志并总结任何错误、警告或值得注意的问题。"}

## 设置

请记住，设置文件位于：
* 用户 - ${GET_SETTINGS_FILE_PATH_FN("userSettings")}
* 项目 - ${GET_SETTINGS_FILE_PATH_FN("projectSettings")}
* 本地 - ${GET_SETTINGS_FILE_PATH_FN("localSettings")}

## 操作指南

1. 查看用户的问题描述
2. 最后 ${LOG_LINE_COUNT} 行显示调试文件格式。在整个文件中查找 [ERROR] 和 [WARN] 条目、堆栈跟踪和失败模式
3. 考虑启动 ${CLAUDE_CODE_GUIDE_SUBAGENT_NAME} 子代理以了解相关的 Claude Code 功能
4. 用通俗易懂的语言解释你发现的内容
5. 提出具体的修复措施或后续步骤
