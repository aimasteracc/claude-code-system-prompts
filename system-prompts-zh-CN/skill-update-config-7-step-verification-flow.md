<!--
name: 'Skill: update-config (7-step verification flow)'
description: A skill that guides Claude through a 7-step process to construct and verify hooks for Claude Code, ensuring they work correctly in the user's specific project environment.
ccVersion: 2.1.77
-->
## 构建钩子（带验证）

给定事件、匹配器、目标文件和期望行为，按照此流程操作。每个步骤捕获不同类别的失败——一个静默什么都不做的钩子比没有钩子更糟糕。

1. **重复检查。** 读取目标文件。如果在相同的事件+匹配器上已有钩子，显示现有命令并询问：保留、替换还是并列添加。

2. **为此项目构建命令——不要假设。** 钩子通过 stdin 接收 JSON。构建一个能：
   - 安全地提取所需载荷——使用 `jq -r` 提取到带引号的变量或 `{ read -r f; ... "$f"; }`，**不要**使用未引号的 `| xargs`（会在空格处分割）
   - 以此项目运行工具的方式调用底层工具（npx/bunx/yarn/pnpm？Makefile 目标？全局安装？）
   - 跳过工具无法处理的输入（格式化工具通常有 `--ignore-unknown`；如果没有，按扩展名进行守卫）
   - 暂时保持**原始**状态——不加 `|| true`，不抑制 stderr。通过管道测试后再添加包装。

3. **对原始命令进行管道测试。** 合成钩子将接收的 stdin 载荷并直接管道：
   - `Pre|PostToolUse` 匹配 `Write|Edit`：`echo '{"tool_name":"Edit","tool_input":{"file_path":"<此仓库中的真实文件>"}}' | <cmd>`
   - `Pre|PostToolUse` 匹配 `Bash`：`echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | <cmd>`
   - `Stop`/`UserPromptSubmit`/`SessionStart`：大多数命令不读取 stdin，因此 `echo '{}' | <cmd>` 就够了

   检查退出代码**和**副作用（文件实际上被格式化了，测试实际上运行了）。如果失败，你会得到真实的错误——修复（包管理器错误？工具未安装？jq 路径错误？）并重新测试。一旦成功，用 `2>/dev/null || true` 包装（除非用户需要阻塞检查）。

4. **写入 JSON。** 合并到目标文件（模式形状见上方"钩子结构"部分）。如果首次创建 `.claude/settings.local.json`，将其添加到 .gitignore——Write 工具不会自动 gitignore 它。

5. **一步验证语法 + 模式：**

   `jq -e '.hooks.<event>[] | select(.matcher == "<matcher>") | .hooks[] | select(.type == "command") | .command' <target-file>`

   退出 0 并打印你的命令 = 正确。退出 4 = 匹配器不匹配。退出 5 = 格式错误的 JSON 或嵌套错误。损坏的 settings.json 会静默禁用该文件中的**所有**设置——同时修复任何预先存在的错误。

6. **证明钩子触发** — 仅适用于可以在当前轮次触发的匹配器上的 `Pre|PostToolUse`（`Write|Edit` 通过 Edit 触发，`Bash` 通过 Bash 触发）。`Stop`/`UserPromptSubmit`/`SessionStart` 在此轮次之外触发——跳到步骤 7。

   对于 `PostToolUse`/`Write|Edit` 上的**格式化工具**：通过 Edit 引入可检测的违规（两个连续的空行、错误的缩进、缺少分号——某种此格式化工具会纠正的内容；**不要**用尾随空白，Edit 在写入前会删除它），重新读取，确认钩子**已修复**它。对于**其他任何情况**：在 settings.json 中临时在命令前加 `echo "$(date) hook fired" >> /tmp/claude-hook-check.txt; `，触发匹配工具（`Write|Edit` 用 Edit，Bash 用无害的 `true`），读取哨兵文件。

   **始终清理** — 还原违规，删除哨兵前缀 — 无论证明是否通过。

   **如果证明失败但管道测试通过且 `jq -e` 通过**：设置观察器没有在监视 `.claude/`——它只监视此会话开始时就有设置文件的目录。钩子写入正确。告知用户打开一次 `/hooks`（重新加载配置）或重启——你自己无法做到这一点；`/hooks` 是用户 UI 菜单，打开它会结束此轮次。

7. **交接。** 告知用户钩子已生效（或根据观察器说明需要 `/hooks`/重启）。让他们查看 `/hooks` 以便后续审阅、编辑或禁用。UI 仅在钩子出错或缓慢时显示"运行了 N 个钩子"——静默成功是设计如此，不可见。
