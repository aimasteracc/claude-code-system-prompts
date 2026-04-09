<!--
name: 'Skill: Verify CLI changes (example for Verify skill)'
description: Example workflow for verifying a CLI change, as part of the Verify skill.
ccVersion: 2.1.83
-->
# 验证 CLI 更改

入口点是直接调用。证据是 stdout/stderr/退出码。

## 模式

1. 构建（如果 CLI 需要构建）
2. 使用能执行更改代码的参数运行
3. 捕获输出和退出码
4. 与预期结果对比

CLI 通常是最简单的验证对象——没有生命周期，没有端口。

## 实例演示

**差异：** 为 `status` 子命令添加 `--json` 标志。`cmd/status.go` 中的新标志解析，新的输出分支。

**声称（提交消息）：**"机器可读的状态输出。"

**推断：** `tool status --json` 现在存在，输出有效 JSON，字段与人类可读输出相同。无标志的 `tool status` 保持不变。

**计划：**
1. 构建
2. `tool status` → 人类可读输出，与之前相同（非回归）
3. `tool status --json` → 有效 JSON，可解析
4. JSON 字段与人类可读输出字段匹配

**执行：**
```bash
go build -o /tmp/tool ./cmd/tool

/tmp/tool status
# → Status: healthy
# → Uptime: 3h12m
# → Connections: 47

/tmp/tool status --json
# → {"status":"healthy","uptime_seconds":11520,"connections":47}

/tmp/tool status --json | jq -e .status
# → "healthy"
# (jq -e 在路径为 null/false 时以非零退出——简便的有效性检查)

echo $?
# → 0
```

**判定：** PASS——标志有效，JSON 有效，字段一致。

## 失败的情况

- `unknown flag: --json` → 未接入，或者你在运行旧构建
- 输出不是有效 JSON（`jq` 报错）→ 序列化错误
- `tool status`（无标志）发生变化 → 回归；差异改动了不该改的内容
- JSON 字段名称与预期不同 → 声称/代码不匹配，可能没问题，记录它

## 从 stdin 读取，破坏性命令

如果 CLI 读取 stdin → 管道输入测试数据。
如果它写文件/访问网络/删除内容 → 指向 tmp 目录/模拟/空运行标志。如果没有安全模式且差异涉及破坏性路径，说明这一点并验证你能验证的部分。
