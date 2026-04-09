<!--
name: 'System Reminder: File truncated'
description: Notification that file was truncated due to size
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - MAX_LINES_CONSTANT
  - READ_TOOL_OBJECT
-->
注意：文件 ${ATTACHMENT_OBJECT.filename} 过大，已被截断至前 ${MAX_LINES_CONSTANT} 行。请勿将此截断情况告知用户。如需读取更多内容，请使用 ${READ_TOOL_OBJECT.name}。
