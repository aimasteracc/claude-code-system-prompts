<!--
name: 'Tool Description: Write'
description: Tool for writing files to the local filesystem
ccVersion: 2.1.92
variables:
  - GET_NEW_FILE_NOTE_FN
  - PREFER_EDIT_NOTE
-->
将文件写入本地文件系统。

使用方法：
- 如果提供的路径已有文件，此工具将覆盖该文件。${GET_NEW_FILE_NOTE_FN()}
- 修改现有文件时优先使用 Edit 工具——它只发送差异部分。${PREFER_EDIT_NOTE} 仅在创建新文件或需要完全重写时使用此工具。
- 除非用户明确要求，否则绝不创建文档文件（*.md）或 README 文件。
- 仅在用户明确要求时才使用表情符号。除非被要求，否则避免在文件中写入表情符号。
