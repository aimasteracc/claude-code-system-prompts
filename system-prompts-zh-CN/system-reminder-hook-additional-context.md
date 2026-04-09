<!--
name: 'System Reminder: Hook additional context'
description: Additional context from a hook
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} 钩子附加上下文：${ATTACHMENT_OBJECT.content.join(`
`)}
