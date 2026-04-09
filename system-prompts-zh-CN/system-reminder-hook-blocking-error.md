<!--
name: 'System Reminder: Hook blocking error'
description: Error from a blocking hook command
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} 钩子命令阻塞错误："${ATTACHMENT_OBJECT.blockingError.command}"：${ATTACHMENT_OBJECT.blockingError.blockingError}
