<!--
name: 'System Reminder: Hook stopped continuation'
description: Message when a hook stops continuation
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} 钩子已停止继续执行：${ATTACHMENT_OBJECT.message}
