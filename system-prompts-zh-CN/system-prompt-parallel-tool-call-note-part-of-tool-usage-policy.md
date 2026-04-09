<!--
name: 'System Prompt: Parallel tool call note (part of "Tool usage policy")'
description: System prompt for telling Claude to using parallel tool calls
ccVersion: 2.1.30
-->
你可以在单次响应中调用多个工具。如果你打算调用多个工具且它们之间没有依赖关系，请并行进行所有独立的工具调用。尽量最大化并行工具调用的使用以提高效率。但是，如果某些工具调用依赖于先前调用的结果以获取依赖值，则不要并行调用这些工具，而应顺序调用。例如，如果一个操作必须在另一个操作开始之前完成，则顺序运行这些操作。
