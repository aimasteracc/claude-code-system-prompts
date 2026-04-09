<!--
name: 'Tool Description: CronCreate'
description: Describes the CronCreate tool for enqueuing one-shot or recurring cron-based jobs with jitter and off-minute scheduling guidance
ccVersion: 2.1.83
variables:
  - CRON_DURABLE_FLAG
  - CANCEL_TIMEFRAME_DAYS
  - CRON_DELETE_TOOL_NAME
-->
将提示词安排在未来某个时间执行。适用于周期性计划和一次性提醒。

使用用户所在时区的标准 5 字段 cron：分钟 小时 日期 月份 星期。"0 9 * * *"表示本地时间上午 9 点——无需时区转换。

## 一次性任务（recurring: false）

对于"在 X 时提醒我"或"在<时间>时执行 Y"的请求——触发一次后自动删除。
将分钟/小时/日期/月份固定为特定值：
  "今天下午 2:30 提醒我检查部署" → cron: "30 14 <today_dom> <today_month> *", recurring: false
  "明天早上运行冒烟测试" → cron: "57 8 <tomorrow_dom> <tomorrow_month> *", recurring: false

## 周期性任务（recurring: true，默认值）

对于"每 N 分钟"/"每小时"/"工作日上午 9 点"的请求：
  "*/5 * * * *"（每 5 分钟），"0 * * * *"（每小时），"0 9 * * 1-5"（工作日本地时间上午 9 点）

## 在任务允许时，避免使用 :00 和 :30 整分钟

每个请求"上午 9 点"的用户都使用 `0 9`，每个请求"每小时"的用户都使用 `0 *`——这意味着来自全球各地的请求同时到达 API。当用户的请求是近似时间时，请选择**不是** 0 或 30 的分钟：
  "每天早上大约 9 点" → "57 8 * * *" 或 "3 9 * * *"（而非 "0 9 * * *"）
  "每小时" → "7 * * * *"（而非 "0 * * * *"）
  "大约一小时后提醒我..." → 选择任意到达时的分钟，不要取整

仅在用户明确指定该时间且含义明确时（"整点 9 点"、"半点"、与会议协调）才使用分钟 0 或 30。有疑问时，稍早或稍晚几分钟——用户不会注意到，而且有助于分散负载。

${CRON_DURABLE_FLAG?`## 持久性

默认情况下（durable: false），任务仅存在于此 Claude 会话中——不写入磁盘，Claude 退出时任务消失。传入 durable: true 可写入 .claude/scheduled_tasks.json，使任务在重启后依然存在。仅在用户明确要求任务持久存在时（"每天继续这样做"、"永久设置"）才使用 durable: true。大多数"5 分钟后提醒我"/"一小时后检查"的请求应保持仅会话有效。`:`## 仅限会话

任务仅存在于此 Claude 会话中——不写入磁盘，Claude 退出时任务消失。`}

## 运行时行为

任务仅在 REPL 空闲时（非查询处理中）触发。${CRON_DURABLE_FLAG?"持久性任务保存到 .claude/scheduled_tasks.json 并在会话重启后存活——下次启动时自动恢复。REPL 关闭期间错过的一次性持久性任务将在下次启动时提示补执行。仅会话任务随进程终止。 ":""}调度器会在你选择的时间基础上添加少量确定性抖动：周期性任务触发最多延迟其周期的 10%（最多 15 分钟）；落在 :00 或 :30 的一次性任务最多提前 90 秒触发。选择非整分钟仍是更大的影响因素。

周期性任务在 ${CANCEL_TIMEFRAME_DAYS} 天后自动过期——最后触发一次后被删除。这限制了会话生命周期。安排周期性任务时请告知用户 ${CANCEL_TIMEFRAME_DAYS} 天的限制。

返回任务 ID，可传给 ${CRON_DELETE_TOOL_NAME}。
