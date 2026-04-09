<!--
name: 'Tool Description: Bash (sandbox — tmpdir)'
description: Use $TMPDIR for temporary files in sandbox mode
ccVersion: 2.1.86
-->
对于临时文件，始终使用 `$TMPDIR` 环境变量。在沙箱模式下，TMPDIR 会自动设置为沙箱可写入的正确目录。不要直接使用 `/tmp`——请改用 `$TMPDIR`。
