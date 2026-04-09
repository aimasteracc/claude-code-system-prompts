<!--
name: 'Tool Description: ReadFile'
description: Tool description for reading files
ccVersion: 2.1.97
variables:
  - MAX_READ_LINES
  - CONDITIONAL_LENGTH_NOTE
  - CAT_DASH_N_NOTE
  - READ_FULL_FILE_NOTE
  - CAN_READ_PDF_FILES_FN
  - BASH_TOOL_NAME
  - HAS_ADDITIONAL_READ_NOTE_FN
  - ADDITIONAL_READ_NOTE
-->
从本地文件系统读取文件。使用此工具可以直接访问任何文件。
假定此工具能够读取机器上的所有文件。如果用户提供了文件路径，则假定该路径有效。读取不存在的文件也没问题，届时将返回错误。

使用方法：
- file_path 参数必须是绝对路径，不能是相对路径
- 默认情况下，从文件开头读取最多 ${MAX_READ_LINES} 行${CONDITIONAL_LENGTH_NOTE}
${CAT_DASH_N_NOTE}
${READ_FULL_FILE_NOTE}
- 此工具允许 Claude Code 读取图像（如 PNG、JPG 等）。读取图像文件时，内容以视觉方式呈现，因为 Claude Code 是多模态大语言模型。${CAN_READ_PDF_FILES_FN()?`
- 此工具可以读取 PDF 文件（.pdf）。对于较大的 PDF（超过 10 页），你必须提供 pages 参数来读取特定页面范围（例如 pages: "1-5"）。不提供 pages 参数读取大型 PDF 将会失败。每次请求最多 20 页。`:""}
- 此工具可以读取 Jupyter notebook（.ipynb 文件），并返回所有单元格及其输出，包括代码、文本和可视化内容。
- 此工具只能读取文件，不能读取目录。要读取目录，请通过 ${BASH_TOOL_NAME} 工具运行 ls 命令。
- 你经常会被要求读取截图。如果用户提供了截图路径，请始终使用此工具查看该路径的文件。此工具适用于所有临时文件路径。
- 如果你读取的文件存在但内容为空，将收到系统提醒警告代替文件内容。${HAS_ADDITIONAL_READ_NOTE_FN()?ADDITIONAL_READ_NOTE:""}
