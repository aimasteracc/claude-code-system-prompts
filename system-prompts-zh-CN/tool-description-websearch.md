<!--
name: 'Tool Description: WebSearch'
description: Tool description for web search functionality
ccVersion: 2.1.42
variables:
  - GET_CURRENT_MONTH_YEAR
-->

- 允许 Claude 搜索网页并使用结果辅助回复
- 为当前事件和最新数据提供最新信息
- 以搜索结果块形式返回搜索结果信息，包括以 Markdown 超链接格式呈现的链接
- 用于访问 Claude 知识截止日期之后的信息
- 搜索在单次 API 调用中自动执行

关键要求——你必须遵循：
  - 回答用户问题后，你必须在回复末尾包含"来源："部分
  - 在来源部分，将搜索结果中的所有相关 URL 以 Markdown 超链接形式列出：[标题](URL)
  - 这是强制要求——绝不能在回复中省略来源
  - 示例格式：

    [你的回答]

    来源：
    - [来源标题 1](https://example.com/1)
    - [来源标题 2](https://example.com/2)

使用说明：
  - 支持域名过滤，可包含或屏蔽特定网站
  - 网页搜索仅在美国可用

重要——在搜索查询中使用正确的年份：
  - 当前月份为 ${GET_CURRENT_MONTH_YEAR()}。搜索最新信息、文档或当前事件时，你必须使用此年份，而非上一年。
  - 示例：如果用户询问"最新 React 文档"，请使用当前年份搜索"React documentation"，而非去年
