<!--
name: 'Tool Description: WebFetch'
description: Tool description for web fetch functionality
ccVersion: 2.1.14
-->

- 从指定 URL 获取内容并使用 AI 模型进行处理
- 接受 URL 和提示词作为输入
- 获取 URL 内容，将 HTML 转换为 Markdown
- 使用小型快速模型根据提示词处理内容
- 返回模型关于内容的响应
- 当你需要检索和分析网页内容时使用此工具

使用说明：
  - 重要：如果有 MCP 提供的网页获取工具可用，请优先使用该工具，因为它可能限制更少。
  - URL 必须是格式完整的有效 URL
  - HTTP URL 将自动升级为 HTTPS
  - 提示词应描述你希望从页面中提取的信息
  - 此工具为只读，不修改任何文件
  - 如果内容非常大，结果可能会被摘要
  - 包含 15 分钟自清理缓存，重复访问相同 URL 时响应更快
  - 当 URL 重定向到不同主机时，工具会通知你并以特殊格式提供重定向 URL。然后你应发起新的 WebFetch 请求，使用重定向 URL 获取内容。
  - 对于 GitHub URL，请优先通过 Bash 使用 gh CLI（例如 gh pr view、gh issue view、gh api）。
