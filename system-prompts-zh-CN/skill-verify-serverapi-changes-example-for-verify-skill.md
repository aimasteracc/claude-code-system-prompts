<!--
name: 'Skill: Verify server/API changes (example for Verify skill)'
description: Example workflow for verifying a server/API change, as part of the Verify skill.
ccVersion: 2.1.83
-->
# 验证服务器/API 更改

入口点是 `curl`（或等效工具）。证据是响应。

## 模式

1. 启动服务器（后台，带就绪轮询——见下文）
2. `curl` 差异涉及的路由，使用能命中更改分支的输入
3. 捕获完整响应（状态 + 标头 + 正文）
4. 与预期结果对比

## 生命周期

如果有运行技能（run-skill），由其处理。如果没有：

```bash
<start-command> &> /tmp/server.log &
SERVER_PID=$!
for i in {1..30}; do curl -sf localhost:PORT/health >/dev/null && break; sleep 1; done
# ... 你的 curl 请求 ...
kill $SERVER_PID
```

没有就绪端点？轮询你即将测试的路由，直到它不再返回连接被拒绝，然后稍微等一下。

## 实例演示

**差异：** 在 `rateLimit.ts` 中为 429 响应添加 `Retry-After` 标头。
**声称（PR 正文）：**"客户端现在可以正确退避。"

**推断：** 触发速率限制后，响应头中应出现 `Retry-After: <n>`。之前没有这个。

**计划：**
1. 启动服务器
2. 足够多次地访问有速率限制的端点以触发 429
3. 检查 429 响应具有 `Retry-After` 标头
4. 检查值是否为正整数

**执行：**
```bash
# 触发限制——10 个快速请求，限制为每秒 5 个（来自差异）
for i in {1..10}; do curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/api/thing; done
# → 200 200 200 200 200 429 429 429 429 429

# 捕获 429 标头
curl -si localhost:3000/api/thing | head -20
# → HTTP/1.1 429 Too Many Requests
# → Retry-After: 12
# → ...
```

**判定：** PASS——`Retry-After: 12` 存在，为正整数。

## 失败的情况

- 标头不存在 → 差异未生效，或者你实际上没有命中 429 路径（先检查状态码）
- 标头存在但值为 `NaN` / `undefined` / 负数 → 逻辑错误
- 所有请求都返回 200 → 你从未触发更改的路径。加紧请求突发或检查速率限制配置。
