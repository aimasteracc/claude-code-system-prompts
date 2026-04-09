<!--
name: 'Skill: Build with Claude API (reference guide)'
description: Template for presenting language-specific reference documentation with quick task navigation
ccVersion: 2.1.91
-->
## 参考文档

以下 `<doc>` 标签中包含你检测到的语言的相关文档。每个标签都有一个 `path` 属性，显示其原始文件路径。使用此路径找到正确的部分：

### 快速任务参考

**单次文本分类/摘要/提取/问答：**
→ 参见 `{lang}/claude-api/README.md`

**聊天 UI 或实时响应显示：**
→ 参见 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间对话（可能超过上下文窗口）：**
→ 参见 `{lang}/claude-api/README.md`——见压缩部分

**提示词缓存/优化缓存/"为什么我的缓存命中率低"：**
→ 参见 `shared/prompt-caching.md` + `{lang}/claude-api/README.md`（提示词缓存部分）

**函数调用/工具使用/代理：**
→ 参见 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批处理（非延迟敏感）：**
→ 参见 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求上传文件：**
→ 参见 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**代理设计（工具接口、上下文管理、缓存策略）：**
→ 参见 `shared/agent-design.md`

**带内置工具的代理（文件/网络/终端）（仅 Python 和 TypeScript）：**
→ 参见 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

**错误处理：**
→ 参见 `shared/error-codes.md`

**通过 WebFetch 获取最新文档：**
→ 参见 `shared/live-sources.md` 中的 URL
