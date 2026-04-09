<!--
name: 'System Reminder: Token usage'
description: Current token usage statistics
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
词元用量：${ATTACHMENT_OBJECT.used}/${ATTACHMENT_OBJECT.total}；剩余 ${ATTACHMENT_OBJECT.remaining}
