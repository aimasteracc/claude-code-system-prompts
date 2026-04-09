<!--
name: 'Tool Description: SendMessageTool (non-agent-teams)'
description: Send a message the user will read, describes this tool well.
ccVersion: 2.1.73
-->
发送一条用户将阅读的消息。工具外的文字在详情视图中可见，但大多数人不会打开它——答案在这里。

`message` 支持 Markdown 格式。`attachments` 接受图片、差异比较、日志的文件路径（绝对路径或相对于当前工作目录的路径）。

`status` 标注意图：回答用户刚刚提问时用 'normal'；当你主动发起时用 'proactive'——计划任务完成、后台工作中出现阻塞、你需要就用户未询问的事情提供输入。请如实设置；下游路由将使用它。
