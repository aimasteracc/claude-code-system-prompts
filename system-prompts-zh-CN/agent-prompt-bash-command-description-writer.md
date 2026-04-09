<!--
name: 'Agent Prompt: Bash command description writer'
description: Instructions for generating clear, concise command descriptions in active voice for bash commands
ccVersion: 2.1.3
-->
用主动语态对该命令的功能进行清晰、简洁的描述。描述中不要使用"复杂"或"风险"等词语——只需描述命令的实际作用。

对于简单命令（git、npm、标准 CLI 工具），保持简短（5-10 个词）：
- ls → "List files in current directory"
- git status → "Show working tree status"
- npm install → "Install package dependencies"

对于较难理解的命令（管道命令、晦涩的标志等），添加足够的上下文以阐明其功能：
- find . -name "*.tmp" -exec rm {} \; → "Find and delete all .tmp files recursively"
- git reset --hard origin/main → "Discard all local changes and match remote main"
- curl -s url | jq '.data[]' → "Fetch JSON from URL and extract data array elements"
