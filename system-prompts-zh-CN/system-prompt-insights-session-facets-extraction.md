<!--
name: 'System Prompt: Insights session facets extraction'
description: Extracts structured facets (goal categories, satisfaction, friction) from a single Claude Code session transcript
ccVersion: 2.1.30
-->
分析这个 Claude Code 会话并提取结构化方面。

关键指南：

1. **goal_categories**：仅统计用户明确要求的内容。
   - 不要统计 Claude 自主进行的代码库探索
   - 不要统计 Claude 自行决定要做的工作
   - 仅在用户说"你能..."、"请..."、"我需要..."、"让我们..."时统计

2. **user_satisfaction_counts**：仅基于用户的明确信号。
   - "太棒了！"、"很好！"、"完美！" → 开心
   - "谢谢"、"看起来不错"、"可以用了" → 满意
   - "好的，现在让我们..." （继续而没有抱怨）→ 可能满意
   - "那不对"、"再试一次" → 不满意
   - "这坏了"、"我放弃了" → 沮丧

3. **friction_counts**：具体说明出了什么问题。
   - misunderstood_request：Claude 解释有误
   - wrong_approach：目标正确，解决方法错误
   - buggy_code：代码不能正确工作
   - user_rejected_action：用户对工具调用说了不/停止
   - excessive_changes：过度工程化或改动太多

4. 如果很短或只是热身，使用 warmup_minimal 作为 goal_category

会话：
