<!--
name: 'Tool Description: Grep'
description: Tool description for content search using ripgrep
ccVersion: 2.0.14
variables:
  - GREP_TOOL_NAME
  - BASH_TOOL_NAME
  - TASK_TOOL_NAME
-->
基于 ripgrep 构建的强大搜索工具

  使用方法：
  - 搜索任务始终使用 ${GREP_TOOL_NAME}。绝不以 ${BASH_TOOL_NAME} 命令方式调用 `grep` 或 `rg`。${GREP_TOOL_NAME} 工具已针对正确的权限和访问进行了优化。
  - 支持完整的正则表达式语法（例如 "log.*Error"、"function\s+\w+"）
  - 使用 glob 参数（如 "*.js"、"**/*.tsx"）或 type 参数（如 "js"、"py"、"rust"）筛选文件
  - 输出模式："content"显示匹配行，"files_with_matches"仅显示文件路径（默认），"count"显示匹配计数
  - 对于需要多轮搜索的开放性任务，使用 ${TASK_TOOL_NAME} 工具
  - 模式语法：使用 ripgrep（而非 grep）——字面量花括号需要转义（在 Go 代码中查找 `interface{}` 时使用 `interface\{\}`）
  - 多行匹配：默认情况下，模式仅在单行内匹配。对于跨行模式（如 `struct \{[\s\S]*?field`），请使用 `multiline: true`
