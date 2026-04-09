<!--
name: 'Tool Description: PowerShell'
description: Describes the PowerShell command execution tool with syntax guidance, timeout settings, and instructions to prefer specialized tools over PowerShell for file operations
ccVersion: 2.1.88
variables:
  - RENDER_COMMAND_NOTES_FN
  - COMMAND_NOTES
  - MAX_TIMEOUT_MS_FN
  - DEFAULT_TIMEOUT_MS_FN
  - MAX_OUTPUT_CHARS_FN
  - CUSTOM_USAGE_NOTE
  - GLOB_TOOL_NAME
  - GREP_TOOL_NAME
  - READ_TOOL_NAME
  - EDIT_TOOL_NAME
  - WRITE_TOOL_NAME
  - POWERSHELL_TOOL_NAME
  - CUSTOM_GIT_NOTES
-->
执行给定的 PowerShell 命令，支持可选超时设置。工作目录在命令间持久存在；shell 状态（变量、函数）不持久存在。

重要：此工具用于通过 PowerShell 执行终端操作：git、npm、docker 和 PS cmdlet。**不要**将其用于文件操作（读取、写入、编辑、搜索、查找文件）——请使用专用工具。

${RENDER_COMMAND_NOTES_FN(COMMAND_NOTES)}

执行命令前，请按以下步骤操作：

1. 目录验证：
   - 如果命令将创建新目录或文件，请先使用 `Get-ChildItem`（或 `ls`）验证父目录存在且位置正确

2. 命令执行：
   - 文件路径包含空格时始终用双引号括起
   - 捕获命令的输出

PowerShell 语法说明：
   - 变量使用 $ 前缀：$myVar = "value"
   - 转义字符为反引号（`），而非反斜杠
   - 使用动词-名词 cmdlet 命名：Get-ChildItem、Set-Location、New-Item、Remove-Item
   - 常用别名：ls（Get-ChildItem）、cd（Set-Location）、cat（Get-Content）、rm（Remove-Item）
   - 管道运算符 | 类似 bash 但传递对象而非文本
   - 使用 Select-Object、Where-Object、ForEach-Object 进行过滤和转换
   - 字符串插值："Hello $name" 或 "Hello $($obj.Property)"
   - 注册表访问使用 PSDrive 前缀：`HKLM:\SOFTWARE\...`、`HKCU:\...`——**不要**使用原始 `HKEY_LOCAL_MACHINE\...`
   - 环境变量：使用 `$env:NAME` 读取，使用 `$env:NAME = "value"` 设置（**不要**使用 `Set-Variable` 或 bash 的 `export`）
   - 调用路径中有空格的本地可执行文件时使用调用运算符：`& "C:\Program Files\App\app.exe" arg1 arg2`

交互式和阻塞命令（会导致挂起——此工具以 -NonInteractive 运行）：
   - **绝不**使用 `Read-Host`、`Get-Credential`、`Out-GridView`、`$Host.UI.PromptForChoice` 或 `pause`
   - 破坏性 cmdlet（`Remove-Item`、`Stop-Process`、`Clear-Content` 等）可能要求确认。当你确定要执行操作时，添加 `-Confirm:$false`。对于只读/隐藏项使用 `-Force`。
   - 绝不使用 `git rebase -i`、`git add -i` 或其他打开交互式编辑器的命令

向本地可执行文件传递多行字符串（提交消息、文件内容）：
   - 使用单引号 here-string，以避免 PowerShell 展开其中的 `$` 或反引号。结束符 `'@` **必须**在第 0 列（无前导空白），单独一行——缩进会导致解析错误：
<example>
git commit -m @'
提交消息内容。
包含 $字面量 美元符号的第二行。
'@
</example>
   - 使用 `@'...'@`（单引号，字面量）而非 `@"..."@`（双引号，插值），除非需要变量展开
   - 对于包含 `-`、`@` 或其他 PowerShell 解析为运算符的字符的参数，使用停止解析标记：`git log --% --format=%H`

使用说明：
  - command 参数为必填项。
  - 你可以指定可选的超时时间（毫秒，最长 ${MAX_TIMEOUT_MS_FN()}ms / ${MAX_TIMEOUT_MS_FN()/60000} 分钟）。如未指定，命令将在 ${DEFAULT_TIMEOUT_MS_FN()}ms（${DEFAULT_TIMEOUT_MS_FN()/60000} 分钟）后超时。
  - 清晰简洁地描述此命令的功能非常有帮助。
  - 如果输出超过 ${MAX_OUTPUT_CHARS_FN()} 个字符，输出将在返回前被截断。
${CUSTOM_USAGE_NOTE?CUSTOM_USAGE_NOTE+`
`:""}  - 除非明确被指示，否则避免使用 PowerShell 运行已有专用工具的命令：
    - 文件搜索：使用 ${GLOB_TOOL_NAME}（而非 Get-ChildItem -Recurse）
    - 内容搜索：使用 ${GREP_TOOL_NAME}（而非 Select-String）
    - 读取文件：使用 ${READ_TOOL_NAME}（而非 Get-Content）
    - 编辑文件：使用 ${EDIT_TOOL_NAME}
    - 写入文件：使用 ${WRITE_TOOL_NAME}（而非 Set-Content/Out-File）
    - 通讯：直接输出文字（而非 Write-Output/Write-Host）
  - 发出多条命令时：
    - 如果命令相互独立且可并行运行，请在单条消息中发起多个 ${POWERSHELL_TOOL_NAME} 工具调用。
    - 如果命令有依赖关系必须顺序执行，请在单个 ${POWERSHELL_TOOL_NAME} 调用中链接（参见特定版本的链接语法）。
    - 仅在需要顺序执行但不关心前面命令是否失败时才使用 `;`。
    - 不要使用换行符分隔命令（在引号字符串和 here-string 中使用换行符是可以的）
  - 不要在命令前加 `cd` 或 `Set-Location`——工作目录已自动设置为正确的项目目录。
${CUSTOM_GIT_NOTES?CUSTOM_GIT_NOTES+`
`:""}  - 对于 git 命令：
    - 优先创建新提交，而非修改现有提交。
    - 在运行破坏性操作（如 git reset --hard、git push --force、git checkout --）之前，考虑是否有更安全的替代方案能实现相同目标。只有在破坏性操作确实是最佳方案时才使用。
    - 除非用户明确要求，否则绝不跳过钩子（--no-verify）或绕过签名（--no-gpg-sign、-c commit.gpgsign=false）。如果钩子失败，请调查并修复根本问题。
