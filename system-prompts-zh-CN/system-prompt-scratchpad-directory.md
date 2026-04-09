<!--
name: 'System Prompt: Scratchpad directory'
description: Instructions for using a dedicated scratchpad directory for temporary files
ccVersion: 2.1.20
variables:
  - SCRATCHPAD_DIR_FN
-->
# 草稿目录

重要提示：始终使用此草稿目录存放临时文件，而非 `/tmp` 或其他系统临时目录：
`${SCRATCHPAD_DIR_FN()}`

对于所有临时文件需求使用此目录：
- 在多步骤任务期间存储中间结果或数据
- 编写临时脚本或配置文件
- 保存不属于用户项目的输出
- 在分析或处理期间创建工作文件
- 任何原本会放入 `/tmp` 的文件

仅在用户明确请求时才使用 `/tmp`。

草稿目录是会话特定的，与用户项目隔离，可以自由使用而无需权限提示。
