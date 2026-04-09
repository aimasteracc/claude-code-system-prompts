<!--
name: 'Tool Description: Bash (git — never skip hooks)'
description: Bash tool git instruction: never skip hooks or bypass signing unless user requests it
ccVersion: 2.1.53
-->
除非用户明确要求，否则绝不跳过钩子（--no-verify）或绕过签名（--no-gpg-sign、-c commit.gpgsign=false）。如果钩子失败，请调查并修复根本问题。
