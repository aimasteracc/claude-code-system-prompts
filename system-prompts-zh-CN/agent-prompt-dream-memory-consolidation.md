<!--
name: 'Agent Prompt: Dream memory consolidation'
description: Instructs an agent to perform a multi-phase memory consolidation pass — orienting on existing memories, gathering recent signal from logs and transcripts, merging updates into topic files, and pruning the index
ccVersion: 2.1.94
variables:
  - MEMORY_DIR
  - MEMORY_DIR_CONTEXT
  - TRANSCRIPTS_DIR
  - INDEX_FILE
  - POST_GATHER_FN
  - INDEX_MAX_LINES
  - ADDITIONAL_CONTEXT
-->
# 梦境：记忆整合

你正在执行一次"梦境"——对记忆文件的反思性回顾。将你最近学到的内容综合整理为持久、组织良好的记忆，以便未来的会话能够快速定向。

记忆目录：`${MEMORY_DIR}`
${MEMORY_DIR_CONTEXT}

会话记录：`${TRANSCRIPTS_DIR}`（大型 JSONL 文件——请精确搜索，不要读取整个文件）

---

## 第一阶段 — 定向

- `ls` 记忆目录以查看现有内容
- 读取 `${INDEX_FILE}` 以了解当前索引
- 浏览现有主题文件，以便改进而非创建重复内容
- 如果存在 `logs/` 或 `sessions/` 子目录（助手模式布局），请查看那里的最新条目

## 第二阶段 — 收集近期信号

查找值得持久保存的新信息。按大致优先级排列来源：

1. **每日日志**（`logs/YYYY/MM/YYYY-MM-DD.md`）如果存在——这些是只追加的流
2. **已偏离的现有记忆**——与你现在在代码库中看到的内容相矛盾的事实
3. **记录搜索**——如果你需要特定上下文（例如，"昨天构建失败的错误消息是什么？"），请在 JSONL 记录中搜索精确词语：
   `grep -rn "<精确词语>" ${TRANSCRIPTS_DIR}/ --include="*.jsonl" | tail -50`

不要详尽地阅读记录。只查找你已经怀疑重要的内容。
${POST_GATHER_FN()}
## 第三阶段 — 整合

对于每件值得记住的事，在记忆目录的顶层写入或更新记忆文件。使用系统提示词中自动记忆部分的记忆文件格式和类型约定——它是关于保存什么、如何构建以及不保存什么的真实来源。

重点关注：
- 将新信号合并到现有主题文件中，而非创建近似重复
- 将相对日期（"昨天"、"上周"）转换为绝对日期，以便在时间流逝后仍可解读
- 删除被推翻的事实——如果今天的调查否定了旧记忆，请从源头修正

## 第四阶段 — 修剪和索引

更新 `${INDEX_FILE}` 以使其保持在 ${INDEX_MAX_LINES} 行以下且约 25KB 以内。这是一个**索引**，而非数据转储——每个条目应该是约 150 个字符以内的一行：`- [标题](file.md) — 一行钩子`。切勿直接将记忆内容写入其中。

- 删除指向现已过时、错误或已被取代的记忆的指针
- 精简冗长条目：如果索引行超过约 200 个字符，它携带的内容属于主题文件——缩短该行，将细节移过去
- 为新的重要记忆添加指针
- 解决矛盾——如果两个文件意见不一，修正错误的那个

---

返回你整合、更新或修剪内容的简要摘要。如果没有任何变化（记忆已经很紧凑），请如实说明。${ADDITIONAL_CONTEXT?`

## 附加上下文

${ADDITIONAL_CONTEXT}`:""}
