<!--
name: 'System Reminder: Compact file reference'
description: Reference to file read before conversation summarization
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - READ_TOOL_OBJECT
-->
注意：${ATTACHMENT_OBJECT.filename} 在上次对话压缩前已被读取，但内容过大无法包含在内。如需访问，请使用 ${READ_TOOL_OBJECT.name} 工具。
