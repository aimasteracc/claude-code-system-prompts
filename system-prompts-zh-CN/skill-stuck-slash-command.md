<!--
name: 'Skill: /stuck slash command'
description: Diagnozse frozen or slow Claude Code sessions
ccVersion: 2.1.77
-->
# /stuck — 诊断卡死/缓慢的 Claude Code 会话

用户认为此机器上的另一个 Claude Code 会话卡死、停滞或运行非常缓慢。进行调查并将报告发布到 #claude-code-feedback。

## 查找对象

扫描其他 Claude Code 进程（排除当前进程——PID 在 `process.pid` 中，但在 shell 命令中只需排除运行此提示词时看到的 PID）。进程名称通常为 `claude`（已安装版本）或 `cli`（本地开发构建）。

卡死会话的迹象：
- **持续高 CPU（≥90%）** — 可能是无限循环。间隔 1-2 秒采样两次以确认不是瞬时峰值。
- **进程状态 `D`（不可中断睡眠）** — 通常是 I/O 挂起。`ps` 输出中的 `state` 列；第一个字符最重要（忽略 `+`、`s`、`<` 等修饰符）。
- **进程状态 `T`（已停止）** — 用户可能意外按了 Ctrl+Z。
- **进程状态 `Z`（僵尸）** — 父进程没有回收子进程。
- **非常高的 RSS（≥4GB）** — 可能存在内存泄漏导致会话缓慢。
- **卡住的子进程** — 挂起的 `git`、`node` 或 shell 子进程可能导致父进程冻结。对每个会话检查 `pgrep -lP <pid>`。

## 调查步骤

1. **列出所有 Claude Code 进程**（macOS/Linux）：
   ```
   ps -axo pid=,pcpu=,rss=,etime=,state=,comm=,command= | grep -E '(claude|cli)' | grep -v grep
   ```
   过滤 `comm` 为 `claude` 或（`cli` 且命令路径包含"claude"）的行。

2. **对于可疑的情况**，收集更多上下文：
   - 子进程：`pgrep -lP <pid>`
   - 如果 CPU 高：1-2 秒后再次采样以确认是持续的
   - 如果子进程看起来挂起（例如 git 命令），使用 `ps -p <child_pid> -o command=` 记录其完整命令行
   - 如果能推断出会话 ID，检查会话的调试日志：`~/.claude/debug/<session-id>.txt`（最后几百行通常显示挂起前的操作）

3. **考虑堆栈转储**用于真正冻结的进程（高级，可选）：
   - macOS：`sample <pid> 3` 给出 3 秒的本地堆栈样本
   - 这很庞大——仅在进程明显挂起且想知道**原因**时才获取

## 报告

**仅在确实发现卡死情况时才发布到 Slack。** 如果所有会话看起来正常，直接告知用户——不要向频道发布全清通知。

如果确实发现了卡死/缓慢的会话，请发布到 **#claude-code-feedback**（频道 ID：`C07VBSHV7EV`），使用 Slack MCP 工具。如果 `slack_send_message` 尚未加载，请使用 ToolSearch 查找。

**使用两条消息结构**以保持频道的可扫描性：

1. **顶层消息** — 一行简短说明：主机名、Claude Code 版本和简洁的症状（例如"会话 PID 12345 持续 100% CPU 达 10 分钟"或"git 子进程在 D 状态下挂起"）。无代码块，无细节。
2. **回帖** — 完整的诊断转储。将顶层消息的 `ts` 作为 `thread_ts` 传入。包括：
   - PID、CPU%、RSS、状态、运行时间、命令行、子进程
   - 你对可能原因的诊断
   - 相关调试日志尾部或 `sample` 输出（如已捕获）

如果 Slack MCP 不可用，将报告格式化为用户可以复制粘贴到 #claude-code-feedback 的消息（并告知他们自行添加回帖中的详情）。

## 说明
- 不要对任何进程发送信号或终止——这仅用于诊断。
- 如果用户提供了参数（例如特定 PID 或症状描述），优先关注该处。
