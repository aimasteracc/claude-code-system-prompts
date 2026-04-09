<!--
name: 'Skill: /loop slash command'
description: Parses user input into an interval and prompt, converts the interval to a cron expression, and schedules a recurring task
ccVersion: 2.1.79
variables:
  - CRON_CREATE_TOOL_NAME
  - DEFAULT_INTERVAL
  - CANCEL_TIMEFRAME_DAYS
  - CRON_DELETE_TOOL_NAME
  - USER_INPUT
-->
# /loop — 安排周期性提示词

将下面的输入解析为 `[间隔] <提示词…>` 并使用 ${CRON_CREATE_TOOL_NAME} 安排。

## 解析（按优先顺序）

1. **前导词元**：如果第一个以空白分隔的词元匹配 `^\d+[smhd]$`（例如 `5m`、`2h`），则作为间隔；其余部分为提示词。
2. **尾随"every"子句**：否则，如果输入以 `every <N><unit>` 或 `every <N> <unit-word>` 结尾（例如 `every 20m`、`every 5 minutes`、`every 2 hours`），提取该内容作为间隔并从提示词中删除。仅在"every"后面是时间表达式时才匹配——`check every PR` 没有间隔。
3. **默认**：否则，间隔为 `${DEFAULT_INTERVAL}`，整个输入为提示词。

如果结果提示词为空，显示用法 `/loop [间隔] <提示词>` 并停止——不调用 ${CRON_CREATE_TOOL_NAME}。

示例：
- `5m /babysit-prs` → 间隔 `5m`，提示词 `/babysit-prs`（规则 1）
- `check the deploy every 20m` → 间隔 `20m`，提示词 `check the deploy`（规则 2）
- `run tests every 5 minutes` → 间隔 `5m`，提示词 `run tests`（规则 2）
- `check the deploy` → 间隔 `${DEFAULT_INTERVAL}`，提示词 `check the deploy`（规则 3）
- `check every PR` → 间隔 `${DEFAULT_INTERVAL}`，提示词 `check every PR`（规则 3——"every"后面没有时间）
- `5m` → 空提示词 → 显示用法

## 间隔 → cron

支持的后缀：`s`（秒，向上取整为最近的分钟，最少 1），`m`（分钟），`h`（小时），`d`（天）。转换：

| 间隔模式 | Cron 表达式 | 说明 |
|---|---|---|
| `Nm` 其中 N ≤ 59 | `*/N * * * *` | 每 N 分钟 |
| `Nm` 其中 N ≥ 60 | `0 */H * * *` | 取整为小时（H = N/60，必须能整除 24） |
| `Nh` 其中 N ≤ 23 | `0 */N * * *` | 每 N 小时 |
| `Nd` | `0 0 */N * *` | 每 N 天午夜本地时间 |
| `Ns` | 视为 `ceil(N/60)m` | cron 最小粒度为 1 分钟 |

**如果间隔无法被其单位整除**（例如 `7m` → `*/7 * * * *` 在 :56→:00 时间隔不均；`90m` → 1.5小时 cron 无法表示），选择最近的整数间隔并在安排前告知用户取整情况。

## 操作

1. 调用 ${CRON_CREATE_TOOL_NAME}，参数为：
   - `cron`：上表中的表达式
   - `prompt`：上面解析出的提示词，原样传递（斜线命令不经修改直接传递）
   - `recurring`：`true`
2. 简要确认：安排的内容、cron 表达式、人类可读的频率、周期性任务在 ${CANCEL_TIMEFRAME_DAYS} 天后自动过期，以及可以通过 ${CRON_DELETE_TOOL_NAME} 提前取消（包含任务 ID）。
3. **然后立即执行解析出的提示词**——不等待第一次 cron 触发。如果是斜线命令，通过 Skill 工具调用；否则直接执行。

## 输入

${USER_INPUT}
