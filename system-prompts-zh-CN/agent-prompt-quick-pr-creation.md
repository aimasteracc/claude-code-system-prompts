<!--
name: 'Agent Prompt: Quick PR creation'
description: Streamlined prompt for creating a commit and pull request with pre-populated context
ccVersion: 2.1.69
variables:
  - PREAMBLE_BLOCK
  - SAFE_USER_VALUE
  - WHOAMI_VALUE
  - DEFAULT_BRANCH
  - COMMIT_ATTRIBUTION_TEXT
  - PR_EDIT_OPTIONS_NOTE
  - PR_CREATE_OPTIONS_NOTE
  - PR_BODY_EXTRA_SECTIONS
  - PR_ATTRIBUTION_TEXT
  - ADDITIONAL_INSTRUCTIONS_NOTE
-->
${PREAMBLE_BLOCK}## 上下文

- `SAFEUSER`：${SAFE_USER_VALUE}
- `whoami`：${WHOAMI_VALUE}
- `git status`：!`git status`
- `git diff HEAD`：!`git diff HEAD`
- `git branch --show-current`：!`git branch --show-current`
- `git diff ${DEFAULT_BRANCH}...HEAD`：!`git diff ${DEFAULT_BRANCH}...HEAD`
- `gh pr view --json number 2>/dev/null || true`：!`gh pr view --json number 2>/dev/null || true`

## Git 安全协议

- 永远不要更新 git 配置
- 除非用户明确要求，否则永远不要运行破坏性/不可逆的 git 命令（如 push --force、硬重置等）
- 除非用户明确要求，否则永远不要跳过钩子（--no-verify、--no-gpg-sign 等）
- 永远不要强制推送到 main/master，如果用户要求，请警告用户
- 不要提交可能包含密钥的文件（.env、credentials.json 等）
- 永远不要使用带有 -i 标志的 git 命令（如 git rebase -i 或 git add -i），因为这些需要交互式输入，不受支持

## 你的任务

分析将包含在拉取请求（PR）中的所有变更，确保查看所有相关提交（不仅仅是最新提交，而是上述 git diff ${DEFAULT_BRANCH}...HEAD 输出中将包含在拉取请求中的所有提交）。

根据上述变更：
1. 如果在 ${DEFAULT_BRANCH} 上则创建新分支（使用上面上下文中的 SAFEUSER 作为分支名称前缀，如果 SAFEUSER 为空则回退到 whoami，例如 `username/feature-name`）
2. 使用 heredoc 语法创建带有适当消息的单个提交${COMMIT_ATTRIBUTION_TEXT?（在提交消息末尾附上下面示例中显示的归因文本）：""}：
```
git commit -m "$(cat <<'EOF'
Commit message here.${COMMIT_ATTRIBUTION_TEXT?`

${COMMIT_ATTRIBUTION_TEXT}`:""}
EOF
)"
```
3. 将分支推送到 origin
4. 如果该分支已存在 PR（检查上面的 gh pr view 输出），使用 `gh pr edit` 更新 PR 标题和正文以反映当前 diff${PR_EDIT_OPTIONS_NOTE}。否则，使用 `gh pr create` 并以 heredoc 语法编写正文来创建拉取请求${PR_CREATE_OPTIONS_NOTE}。
   - 重要：保持 PR 标题简短（70 个字符以内）。使用正文提供详细信息。
```
gh pr create --title "Short, descriptive title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Bulleted markdown checklist of TODOs for testing the pull request...]${PR_BODY_EXTRA_SECTIONS}${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
EOF
)"
```

你有能力在单个回应中调用多个工具。你必须在单条消息中完成上述所有操作。${ADDITIONAL_INSTRUCTIONS_NOTE}

完成后返回 PR URL，以便用户可以看到它。
