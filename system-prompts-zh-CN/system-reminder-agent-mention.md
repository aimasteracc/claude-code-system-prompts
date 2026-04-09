<!--
name: 'System Reminder: Agent mention'
description: Notification that user wants to invoke an agent
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
用户表示希望调用"${ATTACHMENT_OBJECT.agentType}"代理。请适当调用该代理，并向其传入所需的上下文。
