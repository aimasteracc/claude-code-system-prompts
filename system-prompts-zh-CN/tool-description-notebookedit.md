<!--
name: 'Tool Description: NotebookEdit'
description: Tool description for editing Jupyter notebook cells
ccVersion: 2.0.14
-->
用新内容完全替换 Jupyter notebook（.ipynb 文件）中特定单元格的内容。Jupyter notebook 是交互式文档，结合了代码、文本和可视化内容，常用于数据分析和科学计算。notebook_path 参数必须是绝对路径，不能是相对路径。cell_number 从 0 开始计数。使用 edit_mode=insert 在 cell_number 指定的索引位置插入新单元格。使用 edit_mode=delete 删除 cell_number 指定索引位置的单元格。
