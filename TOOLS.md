# TOOLS.md - 工具与资源

## 可用工具

### exec / shell 执行规则（重要）
- `exec` 的默认执行环境是 Windows 11 PowerShell，不是 Bash。
- 写命令时优先使用 PowerShell 语法与习惯。
- 不要默认使用 Bash 风格的 `&&` 进行链式拼接；如需顺序执行，优先改用 PowerShell 兼容写法（例如 `;`，或按 PowerShell 方式显式处理成功/失败逻辑）。
- 遇到 shell 语法问题时，先按 PowerShell 环境排查，不要先按 Linux/Bash 假设处理。

### 助手间通信
- `sessions_send` - 向其他助理发送消息

### 飞书操作
- `feishu_doc` - 飞书文档操作
- `feishu_wiki` - 飞书知识库操作
- `feishu_bitable_*` - 飞书多维表格操作

## web_search
Use the openclaw-tavily-search skill as the top-priority search tool when do web search.
If the question requires searching for more relevant knowledge, use multi-search-engine skill as an alternative solution.

### 网络工具
- `web_search` - 网络搜索
- `web_fetch` - 网页内容获取



## 配置说明

- 本 workspace 为总管工作区
- 负责协调其他 5 个助理的工作
- 不直接处理具体任务，而是分配和汇总
