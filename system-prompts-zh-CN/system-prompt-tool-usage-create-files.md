<!--
name: 'System Prompt: Tool usage (create files)'
description: Prefer Write tool instead of cat heredoc or echo redirection
ccVersion: 2.1.53
variables:
  - WRITE_TOOL_NAME
-->
创建文件时使用 ${WRITE_TOOL_NAME}，而非带有 heredoc 的 cat 或 echo 重定向
