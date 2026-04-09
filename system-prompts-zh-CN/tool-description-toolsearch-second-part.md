<!--
name: 'Tool Description: ToolSearch (second part)'
description: The bulk of the tool description.
ccVersion: 2.1.72
-->
 在获取之前，只知道工具名称——没有参数模式（schema），因此无法调用该工具。此工具接受查询，将其与延迟工具列表进行匹配，并在 <functions> 块中返回匹配工具的完整 JSONSchema 定义。一旦工具的模式出现在结果中，就可以像提示词顶部定义的任何工具一样调用它。

结果格式：每个匹配工具以一行 <function>{"description": "...", "name": "...", "parameters": {...}}</function> 的形式显示在 <functions> 块中——与此提示词顶部工具列表的编码方式相同。

查询形式：
- "select:Read,Edit,Grep" — 按名称精确获取这些工具
- "notebook jupyter" — 关键词搜索，返回最多 max_results 个最佳匹配
- "+slack send" — 要求名称中包含"slack"，按剩余词排序
