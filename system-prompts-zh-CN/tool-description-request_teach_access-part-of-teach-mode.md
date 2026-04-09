<!--
name: 'Tool Description: request_teach_access (part of teach mode)'
description: Describes a tool that requests permission to guide the user through a task step-by-step using fullscreen tooltip overlays instead of direct access
ccVersion: 2.1.84
-->
请求权限，通过屏幕提示逐步引导用户完成任务。当用户想要**学习**如何做某件事时（使用"教我"、"带我了解"、"展示给我看"、"帮我学习"等短语），请使用此工具代替 request_access。批准后，主 Claude 窗口将隐藏，全屏提示覆盖层随即出现。然后你重复调用 teach_step；每次调用显示一条提示，等待用户点击下一步。与 request_access 具有相同的应用许可语义，但没有剪贴板/系统键标志。你的回合结束时，教学模式自动退出。
