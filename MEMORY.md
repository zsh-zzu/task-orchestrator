# Memory - 贾维斯 (总管)

## Obsidian 笔记库规则

**路径**：`E:\software\obsidian_github_store\Obsidian`

**规则**：当张博说"放到 Obsidian 中"时，所有 .md 文件都放到这个目录下。

**记录时间**：2026-03-11

---

## 重要资源

### OpenClaw Use Cases 仓库
**链接**：https://github.com/hesamsheikh/awesome-openclaw-usecases

**简介**：社区收集的 OpenClaw 真实用例，解决 OpenClaw 适配的瓶颈问题。

**分类用例**：

| 类别 | 用例 | 适合谁 |
|------|------|--------|
| **内容消费** | Daily Reddit Digest, YouTube Digest, Tech News Digest | 小艺 |
| **内容创作** | YouTube Content Pipeline, Multi-Agent Content Factory, Game Dev Pipeline | 小艺、auto |
| **基础设施** | n8n Workflow Orchestration, Self-Healing Home Server | auto |
| **生产力** | Project Management, Customer Service, Phone Assistant, Meeting Notes | 全员 |
| **知识管理** | Knowledge Base RAG, Semantic Memory Search, Second Brain | 老铁 |
| **金融** | AI Earnings Tracker, Polymarket Autopilot | stock、bo |
| **健康生活** | Health Tracker, Habit Tracker, Family Calendar | mate |

**特别推荐的用例**：
- Multi-Agent Specialized Team - 多专家团队协作（类似我们的架构）
- Self-Improving Agent - 持续改进（已安装）
- Semantic Memory Search - 语义记忆搜索
- Goal-Driven Autonomous Tasks - 目标驱动自主任务

**记录时间**：2026-03-10

---

## 网络搜索规则（2026-03-22）⚠️ 重要

**禁止**：使用内置 `web_search` 工具（Brave Search API）

**允许**：使用已安装的搜索 skill
- `multi-search-engine`（推荐，17个搜索引擎）
- `exa-web-search-free`（免费）
- `tavily-search`

**适用范围**：全体团队成员

---

## Memory 功能配置（2026-03-22）

| 功能 | 状态 | 说明 |
|------|------|------|
| memory_search | ✅ 可用 | 本地 embedding，Vulkan GPU 加速 |
| memory_get | ✅ 可用 | 直接读取文件 |
| Provider | local | 不需要 API key |

**模型**：embeddinggemma-300m-qat-Q8_0.gguf (328MB)

**注意**：
- 智谱 Coding Plan **不包含** embedding API（需单独充值）
- semantic-memory skill 已删除

---

## 重要规则

### Chrome 浏览器连接规则（2026-03-23 更新）⚠️ 重要

**状态：已废弃（2026-03-28）**

以下旧规则已不再适用：通过 OpenClaw Browser Relay / `profile="chrome-relay"` 连接浏览器、要求用户把 Browser Relay 打开到 ON/绿色、禁止使用 `profile="user"`。

**废弃原因**：
- 当前 OpenClaw 官方文档中的浏览器 profile 以 `user` / `openclaw` 为主
- 实测当前环境可用 profile 只有 `user` 和 `openclaw`
- `chrome-relay` 已不在当前环境中可用，继续沿用会误导排查

**新的结论（2026-03-28）**：
- 优先使用 `profile="user"` 或 `profile="openclaw"`
- 不再把“Browser Relay 图标 ON/绿色”当作浏览器可用的默认前提
- 若 `user` profile 失败，应排查 existing-session / chrome-mcp / remote debugging / `DevToolsActivePort` 等问题
- 以当前官方文档和实际 `browser profiles` 输出为准

**用户新要求（2026-03-28）**：
- 以后全体助理默认使用张博自己的 Chrome 浏览器
- 不要擅自新建新的 Chrome 实例
- 浏览器操作优先走用户现有 Chrome 会话（优先 `profile="user"`）
- 只有在用户明确允许或任务确有必要时，才考虑独立浏览器实例

**适用范围**：全体团队成员（leader、scholar、auto、creator、stock、mate、bian）

**原记录时间**：2026-03-23
**废弃时间**：2026-03-28
**备注**：保留此段仅用于提醒“旧规则已失效”，避免后续再次误用。

---

### 助理权限优化（2026-03-23）⚠️ 重要

**问题**：stock 助理误判 Chat history，自主调用 `sessions_send` 协调团队

**解决方案**：为所有助理创建 `SYSTEM_RESTRICTIONS.md` 文件

**核心规则**：
1. ✅ **可以主动协调**：当用户明确要求或工作需要时
2. ✅ 只处理直接 @ 自己的消息
3. ✅ Chat history 中的其他助理消息仅供参考
4. ✅ 需要协调时合理判断并向用户说明
5. ✅ **连续 5 次失败必须换思路**：同一件事、同一路径、同一种方法连续尝试 5 次仍未成功，必须停止硬试，改用新方法、换排查方向、降级验证，或向用户/leader 说明卡点请求决策

**创建的文件**：
- `workspace-stock/SYSTEM_RESTRICTIONS.md`
- `workspace-scholar/SYSTEM_RESTRICTIONS.md`
- `workspace-auto/SYSTEM_RESTRICTIONS.md`
- `workspace-creator/SYSTEM_RESTRICTIONS.md`
- `workspace-mate/SYSTEM_RESTRICTIONS.md`
- `workspace-bian/SYSTEM_RESTRICTIONS.md`

**确认状态**：scholar ✅，其他助理下次激活会看到

**记录时间**：2026-03-23；2026-03-25 更新

### Memory 操作规则
**规则**：不准直接添加/修改别人的 memory 文件，只能告诉其他成员让他们自己添加。每个人只有更新自己 memory 的权限。

**例外**：贾维斯可以自己更新自己的 memory。

**记录时间**：2026-03-09

### JSON 配置文件格式规范（2026-03-25）

**规则**：所有 JSON 配置文件必须使用标准格式（2个空格缩进）

**格式化方法**：
```python
# 使用 Python 格式化（推荐）
python -c "import json; data = json.load(open('file.json', 'r', encoding='utf-8')); open('file.json', 'w', encoding='utf-8').write(json.dumps(data, indent=2, ensure_ascii=False))"
```

**禁止**：
- ❌ 使用 PowerShell 的 `ConvertTo-Json`（默认缩进 17 个空格，格式混乱）
- ❌ 使用 4 空格或 tab 缩进
- ❌ 手动调整缩进（容易出错）

**验证**：
```python
python -c "import json; json.load(open('file.json', 'r', encoding='utf-8')); print('JSON 格式验证通过 ✓')"
```

**适用范围**：
- `openclaw.json`
- `openclaw - 副本.json`（备份）
- 所有助理 workspace 中的配置文件
- 任何需要人工编辑的 JSON 文件

**记录时间**：2026-03-25

### 升级操作需先汇报
**规则**：以后升级（如 npm upgrade、系统更新等）这种重要事情，必须先向张博汇报，得到允许后再执行。

**原因**：升级操作可能影响系统稳定性，需要张博确认后才能进行。

**记录时间**：2026-03-09

---

## 错误反思：安装 OpenClaw 插件的正确流程（2026-03-21）

### 我犯的错误
在安装 `openclaw-semantic-memory` 时，我犯了以下错误：

#### 1. 没有先仔细阅读官方文档
**错误行为**：
- 看到 ClawHub skill 就直接安装
- 没有先查看 README 了解安装要求
- 没有检查依赖、Node.js 版本等前置条件

**正确做法**：
- ✅ 安装任何插件/skill 前必须先阅读 README.md
- ✅ 检查是否有特殊依赖、版本要求、构建工具要求
- ✅ 确认安装路径（全局插件 vs workspace skills）

#### 2. 安装位置错误
**错误行为**：
- 安装到 `workspace/skills/openclaw-semantic-memory/`
- 后来才发现应该安装到全局 `~/.openclaw/plugins/semantic-memory/`

**正确做法**：
- ✅ OpenClaw 插件应安装到 `~/.openclaw/plugins/` 目录
- ✅ Workspace skills 是给特定 agent 用的，不是全局插件

#### 3. 没有先安装依赖
**错误行为**：
- 安装完 skill 就想直接配置启用
- 忘记运行 `npm install` 安装依赖包

**正确做法**：
- ✅ 安装插件后必须先运行 `npm install`
- ✅ 检查 `package.json` 确认依赖是否完整安装
- ✅ 如有原生模块，可能需要额外的构建工具

#### 4. 急于修改配置文件
**错误行为**：
- 在确认插件能正常工作前就修改 `openclaw.json`
- 导致配置错误，插件无法加载

**正确做法**：
- ✅ **先测试，后配置**
- ✅ 确认依赖安装完成
- ✅ 确认插件能被 OpenClaw 识别（`openclaw plugins list`）
- ✅ 最后才修改配置文件

#### 5. 没有一口气完成所有步骤
**错误行为**：
- 分多次尝试，让用户等待
- 没有一次性告知所有必要步骤

**正确做法**：
- ✅ 一次性完成：安装 → 依赖 → 配置 → 验证
- ✅ 在开始前就列出完整步骤清单
- ✅ 避免"试一步看一步"的低效方式

---

### 正确的插件安装流程（标准化）

以后安装任何 OpenClaw 插件/skill，必须遵循以下流程：

```
1. 阅读文档
   ├─ 读取 README.md
   ├─ 检查 package.json（依赖、Node 版本）
   └─ 确认安装路径要求

2. 安装前置检查
   ├─ Node.js 版本是否符合？
   ├─ 是否需要构建工具（Windows Build Tools）？
   └─ 是否需要网络访问（下载模型）？

3. 安装插件
   ├─ ClawHub: `clawhub install <skill-name>`
   ├─ 复制到正确位置（全局 vs workspace）
   └─ 运行 `npm install` 安装依赖

4. 验证安装
   ├─ 检查 node_modules 是否存在
   ├─ 运行 `openclaw plugins list` 确认识别
   └─ 如有测试命令，先运行测试

5. 修改配置
   ├─ 只在确认插件可用后才修改配置
   ├─ 修改 `openclaw.json` 启用插件
   └─ 重启 gateway

6. 最终验证
   ├─ 检查插件是否加载（`openclaw plugins list`）
   ├─ 测试插件功能
   └─ 记录安装结果
```

---

### 本次教训总结

1. **文档优先** - 任何新东西，先读文档再动手
2. **依赖优先** - 安装任何东西，先处理依赖
3. **验证优先** - 修改配置前，先验证可用性
4. **全局视角** - 一次性规划所有步骤，不要走一步看一步

**记录时间**：2026-03-21 16:57
**严重程度**：⭐⭐⭐⭐⭐（浪费了大量时间和用户耐心）

---

## sessions_send 分批发送规则（2026-03-21）

**问题**：一次性向多个助理发送 `sessions_send` 会导致队列堵塞，任务等待时间过长（超时）。

**规则**：以后分批发送，避免队列堵塞。

**分批策略**：
1. **每批最多 2-3 个助理**
2. **批次间隔 10-15 秒**
3. **优先级排序**：紧急任务优先，非紧急任务延后

**示例**：
```python
# ❌ 错误：一次性发送 6 个
for agent in [scholar, auto, creator, stock, mate, bian]:
    sessions_send(agentId=agent, message="...")

# ✅ 正确：分批发送
# 第 1 批
sessions_send(agentId="scholar", message="...")
sessions_send(agentId="auto", message="...")
# 等待 10-15 秒

# 第 2 批
sessions_send(agentId="creator", message="...")
sessions_send(agentId="stock", message="...")
# 等待 10-15 秒

# 第 3 批
sessions_send(agentId="mate", message="...")
sessions_send(agentId="bian", message="...")
```

**记录时间**：2026-03-21

---

## 今日总结 (2026-03-22)

### Memory 功能修复完成
1. **安装 node-llama-cpp@3.16.2** - 本地 embedding 支持
2. **配置 local provider** - 使用 Vulkan GPU 加速
3. **下载模型** - embeddinggemma-300m-qat-Q8_0.gguf (328MB)
4. **删除 semantic-memory skill** - 已从所有助理 workspace 清理

### 网络搜索规则更新
- **禁用** web_search (Brave API) - 未配置 API key
- **使用** multi-search-engine / exa-web-search-free / tavily-search

### 通知状态
| 助理 | 状态 |
|------|------|
| scholar | ✅ 已确认整理 |
| auto | ⏳ 超时 |
| creator | ⏳ 超时 |
| stock | ⏳ 超时 |
| mate | ⏳ 超时 |
| bian | ⏳ 超时 |

---

### 我学到的东西

1. **Obsidian 笔记库路径**
   - 路径：`E:\software\obsidian_github_store\Obsidian`
   - 规则：当张博说"放到 Obsidian 中"时，.md 文件存到这个目录

2. **小红书无货源开店全流程**
   - 完整流程已保存到 `memory/xiaohongshu-business.md`
   - 千帆后台：https://ark.xiaohongshu.com/app-system/home
   - 0粉可开店，相当于2020年的抖音小店

3. **CLI-Anything 工具**
   - 仓库：https://github.com/HKUDS/CLI-Anything
   - 一行命令让任何软件生成 CLI 接口供智能体调用
   - 对自动化非常有价值

4. **Windows 上 Tavily 搜索**
   - 要用 `python` 而不是 `python3`

5. **OpenClaw 浏览器工具**
   - 自带 Tab 管理功能：open/tabs/focus/close/snapshot/act

### 我犯的错误

1. **搜索闲鱼规则时不够诚实**
   - 问题：搜索工具失败后，第一次回答没有明确说明是基于常识
   - 教训：当无法获取实时数据时，必须第一时间告诉用户

2. **子代理搜索超时**
   - 问题：派子代理搜索也失败了
   - 原因：浏览器工具超时
   - 教训：Windows 上有些工具需要调整


## API 密钥管理

### 智谱 coding plan API
```
f752e5ce70fa40fe86d84ec4dd525a45.0f4MnqoqDaTlqq8K
```
### 火山方舟 coding plan API
```
8bc78212-818a-495b-b3b9-b2d0021e1af9
```
**规则**：其他成员需要时再提供给他们，不要泄露给无关人员。

---

## 飞书错误处理方案

**问题**：当回复的消息被用户撤回时，飞书 API 返回错误码 230011 ("The message was withdrawn")

**解决方案**：代码需要增加对 230011 错误的处理，检测到"消息已撤回"时直接跳过，不再重试。

**说明**：OpenClaw 的 send.ts 已经有处理逻辑（WITHDRAWN_REPLY_ERROR_CODES），会 fallback 到直接发送消息而不是回复。

---

## 浏览器使用规则（2026-03-23 更新）

### 连接方式
**必须使用 Browser Relay**：`profile="chrome-relay"`

### 强制规则（全体助理遵守）
1. **连接方式**：所有浏览器操作必须使用 `profile="chrome-relay"`，禁止其他方式
2. **Tab管理**：严格管理浏览器标签页，不用的标签页及时关闭，避免资源浪费
3. **搜索要求**：搜索时必须使用多源搜索，除了常规搜索外，必须同时使用Google等境外搜索引擎查询相关信息

### 触发条件
当张博提到以下关键词时，都使用 Browser Relay 方式：
- "浏览器" / "chrome" / "Chrome"
- "打开网页" / "访问网站"
- "网页操作" / "自动化"

**适用范围**：所有需要浏览器操作的任务，包括搜索、网页访问、自动化操作等
**记录时间**：2026-03-23（替代 2026-03-13 的旧规则）

---

## 老铁改进模型注意事项（2026-03-13）

**目标模型**：mymode/GSDANet_base_DyHighPassAttnMaskV2Light_CAHighPass_CrosSpaAttnOldDownUp012502040608_c24.py

**分析要求**：
1. 只看 if __name__ == "__main__": 中真正用到的class
2. 被注释掉的代码全部忽略
3. 编码阶段：目前只在后两层用了Res_CBAM_block_hpf_cahpdwt，需要分析如何调整
4. 跳跃连接阶段：目前是照搬别人的，需要根据红外小目标检测特点改进

**记录时间**：2026-03-13

---

## Register 仓库（2026-03-13）

**仓库地址**：https://github.com/MasterAlanLab/Register

**功能**：自动化批量注册脚本集合
- openai-register：OpenAI自动注册
- grok-register：Grok(x.ai)自动注册
- tavily-register：Tavily API自动注册

**成本**：需要GPTMail API Key + YesCaptcha Key + 代理（非CN/HK）

**风险**：批量注册可能违反平台ToS，有封号风险

**记录时间**：2026-03-13

---

## OpenAI 注册脚本测试结果（2026-03-13）

**测试状态**：脚本运行成功，但GPTMail共享Key配额用完

**IP所在地**：SG（新加坡）- 满足OpenAI注册要求

**阻塞原因**：GPTMail API共享Key `gpt-test` 每日20万次配额已用完

**解决方案**：
1. LDC Store获取100次免费测试Key（https://shop.chatgpt.org.uk）
2. 添加自己的域名获取20000次额度
3. 等待明天配额重置

**仓库位置**：`D:\.openclaw\workspace-leader\Register`

**记录时间**：2026-03-13

---

## 战略调整（2026-03-13 周五）

**决策**：不做卖人的东西，改做卖宠物等的生意

**项目变更**：
- 唇膏项目：作废，需要回收

**记录时间**：2026-03-13
