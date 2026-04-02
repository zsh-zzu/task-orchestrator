# Memory - 贾维斯 (总管)

## ⚠️ 重要规则

### Chrome 浏览器连接规则（2026-03-29）

**核心原则：永远不要主动关闭张博的 Chrome！**

1. ✅ 直接用 `profile="user"` 尝试连接
2. ✅ 连接成功后，先 `browser(action="tabs")` 查看现有标签页
3. ❌ 连不上时，不要尝试关闭或重启 Chrome
4. ✅ 连不上时，提示张博手动操作：
   - 关闭所有 Chrome 窗口
   - 用命令启动：`Start-Process "chrome.exe" -ArgumentList "--remote-debugging-port=9222"`
5. ❌ 不要用 `chrome-relay`
6. ✅ 不要随意关闭标签页，不要在张博正在浏览的页面上操作

**适用范围**：全体团队成员

### 网络搜索规则（2026-03-22）

**禁止**：内置 `web_search`（Brave Search API）
**允许**：`multi-search-engine`（推荐）、`exa-web-search-free`、`tavily-search`

### Memory 操作规则（2026-03-09）

不准直接添加/修改别人的 memory 文件。贾维斯只更新自己的 memory。

### 升级操作需先汇报（2026-03-09）

npm upgrade、系统更新等必须先向张博汇报，得到允许后再执行。

### sessions_send 分批发送（2026-03-21）

每批最多 2-3 个助理，批次间隔 10-15 秒，避免队列堵塞。

### 子任务完成通知规则（2026-04-01）

**核心**：收到 subagent announce（sourceChannel=webchat）时，**必须用 `message(action="send")` 显式发送到 Telegram**，不能只靠文字回复（文字回复会路由到 webchat 内部，张博看不到）。

步骤：
1. 收到 announce → 用 `message(action="send", channel="telegram", message="...")` 发送通知
2. 回复 `NO_REPLY`（避免重复）

### 连续失败换思路（2026-03-25）

同一件事连续 5 次失败，必须停止，换方法或向用户说明卡点。

### JSON 配置文件格式（2026-03-25）

2 空格缩进，用 Python `json.dumps(indent=2)` 格式化，禁止 `ConvertTo-Json`。

### Skill 开发安全规范（2026-03-31）

开发 skill 时示例和文档中**禁止暴露**：
- 具体 agent ID（leader/scholar/auto/stock/creator/mate/bian）
- 团队角色映射表
- 用户个人信息（姓名、GitHub 用户名、Telegram ID 等）
统一使用通用占位符（orchestrator/research-agent/dev-agent 等）。

### 助理权限优化（2026-03-23）

各助理 workspace 均有 `SYSTEM_RESTRICTIONS.md`：
- 只处理直接 @ 自己的消息
- Chat history 中其他助理消息仅供参考
- 需要协调时合理判断并向用户说明

---

## Git 仓库

| 仓库 | 本地路径 | GitHub Remote |
|------|----------|---------------|
| workspace-leader | `D:\.openclaw\workspace-leader` | ⚠️ **待配置** |
| Register | `D:\.openclaw\workspace-leader\Register` | https://github.com/MasterAlanLab/Register |

## 常用路径

| 名称 | 路径 |
|------|------|
| Obsidian 笔记库 | `E:\software\obsidian_github_store\Obsidian` |

**Obsidian 规则**：张博说"放到 Obsidian 中"时，.md 文件存到该目录。

---

## Memory 功能（2026-03-22）

- memory_search ✅ | memory_get ✅ | local provider
- 模型：embeddinggemma-300m-qat-Q8_0.gguf (328MB)
- 智谱 Coding Plan 不含 embedding API

---

## API 密钥

- 智谱 coding plan：`f752e5ce70fa40fe86d84ec4dd525a45.0f4MnqoqDaTlqq8K`
- 火山方舟 coding plan：`8bc78212-818a-495b-b3b9-b2d0021e1af9`
- 其他成员需要时再提供，不要泄露

---

## 插件安装标准流程（2026-03-21 教训总结）

1. **读文档** → README.md、package.json
2. **前置检查** → Node 版本、构建工具
3. **安装** → clawhub install / 手动复制 + npm install
4. **验证** → openclaw plugins list 确认识别
5. **配置** → 确认可用后才改 openclaw.json
6. **最终验证** → 重启 gateway + 测试功能

**教训**：文档优先、依赖优先、验证优先、全局视角

---

## 飞书错误处理

消息被撤回时飞书 API 返回 230011，OpenClaw send.ts 已有 WITHDRAWN_REPLY_ERROR_CODES 处理。

---

## 项目与资源

### OpenClaw Use Cases
https://github.com/hesamsheikh/awesome-openclaw-usecases — 社区用例集合

### Register 仓库
https://github.com/MasterAlanLab/Register — 自动化批量注册脚本（OpenAI/Grok/Tavily）
风险：可能违反平台 ToS

### 老铁改进模型（2026-03-13）
- 目标：GSDANet 红外小目标检测
- 只看 `if __name__ == "__main__":` 中真正用到的 class
- 跳跃连接需根据红外小目标检测特点改进

### 小红书无货源开店
- 千帆后台：https://ark.xiaohongshu.com/app-system/home
- 0粉可开店

### CLI-Anything
https://github.com/HKUDS/CLI-Anything — 让任何软件生成 CLI 接口

### 战略调整（2026-03-13）
不做卖人的东西，改做卖宠物等的生意；唇膏项目作废
