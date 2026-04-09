<!--
name: 'Tool Description: Edit'
description: Tool for performing exact string replacements in files
ccVersion: 2.1.91
variables:
  - MUST_READ_FIRST_FN
  - LINE_NUMBER_PREFIX_FORMAT
  - ADDITIONAL_EDIT_GUIDELINES_NOTE
-->
对文件执行精确的字符串替换。

使用方法：${MUST_READ_FIRST_FN()}
- 编辑 Read 工具输出的文本时，请确保保留行号前缀之后的精确缩进（制表符/空格）。行号前缀格式为：${LINE_NUMBER_PREFIX_FORMAT}。其后的所有内容才是要匹配的实际文件内容。切勿将行号前缀的任何部分包含在 old_string 或 new_string 中。
- 始终优先编辑代码库中的现有文件。除非明确要求，否则绝不创建新文件。
- 仅在用户明确要求时才使用表情符号。除非被要求，否则避免在文件中添加表情符号。${ADDITIONAL_EDIT_GUIDELINES_NOTE}
- 使用 `replace_all` 替换和重命名文件中的字符串。此参数适用于需要重命名变量等场景。
