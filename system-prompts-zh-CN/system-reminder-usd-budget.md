<!--
name: 'System Reminder: USD budget'
description: Current USD budget statistics
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
美元预算：$${ATTACHMENT_OBJECT.used}/$${ATTACHMENT_OBJECT.total}；剩余 $${ATTACHMENT_OBJECT.remaining}
