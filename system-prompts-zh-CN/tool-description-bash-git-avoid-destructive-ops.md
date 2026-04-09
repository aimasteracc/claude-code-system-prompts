<!--
name: 'Tool Description: Bash (git — avoid destructive ops)'
description: Bash tool git instruction: consider safer alternatives to destructive operations
ccVersion: 2.1.53
-->
在运行破坏性操作（如 git reset --hard、git push --force、git checkout --）之前，考虑是否有更安全的替代方案能实现相同目标。只有在破坏性操作确实是最佳方案时才使用。
