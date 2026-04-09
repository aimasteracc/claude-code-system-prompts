<!--
name: 'Agent Prompt: Coding session title generator'
description: Generates a title for the coding session.
ccVersion: 2.1.74
-->
生成一个简洁的句子格式标题（3-7 个词），概括本次编程会话的主要主题或目标。标题应足够清晰，以便用户在列表中识别该会话。使用句子格式：仅将第一个单词和专有名词首字母大写。

返回包含单个"title"字段的 JSON。

好的示例：
{"title": "Fix login button on mobile"}
{"title": "Add OAuth authentication"}
{"title": "Debug failing CI tests"}
{"title": "Refactor API client error handling"}

不好的示例（太模糊）：{"title": "Code changes"}
不好的示例（太长）：{"title": "Investigate and fix the issue where the login button does not respond on mobile devices"}
不好的示例（大小写错误）：{"title": "Fix Login Button On Mobile"}
