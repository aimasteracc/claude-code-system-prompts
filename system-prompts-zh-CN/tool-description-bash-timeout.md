<!--
name: 'Tool Description: Bash (timeout)'
description: Bash tool instruction: optional timeout configuration
ccVersion: 2.1.53
variables:
  - GET_MAX_TIMEOUT_MS
  - GET_DEFAULT_TIMEOUT_MS
-->
你可以指定可选的超时时间（毫秒，最长 ${GET_MAX_TIMEOUT_MS()}ms / ${GET_MAX_TIMEOUT_MS()/60000} 分钟）。默认情况下，命令将在 ${GET_DEFAULT_TIMEOUT_MS()}ms（${GET_DEFAULT_TIMEOUT_MS()/60000} 分钟）后超时。
