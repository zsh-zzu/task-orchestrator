# Task Orchestrator 最佳实践

## 1. 任务分解最佳实践

### 粒度判断原则
- **单一职责**：每个任务只做一件事，可独立验证
- **时间盒**：理想任务 5-30 分钟可完成，超过 1 小时应拆分
- **不过度拆分**：如果两个任务总有相同的依赖关系、分配给同一个 Agent，合并它们

```
❌ 过度拆分：打开文件 → 读取内容 → 解析JSON → 提取字段（合并为"解析配置文件"）
❌ 粒度过粗：完成整个后端系统（应拆为 API设计、数据库、认证、业务逻辑等）
✅ 适中粒度：设计用户认证 API 端点并编写单元测试
```

### 依赖关系标记
- 使用 `depends_on: [task_id]` 显式声明
- 依赖方向应为 DAG（有向无环图），避免循环依赖
- 跨 Agent 的数据传递必须标记依赖

### 并行任务识别
- **无数据依赖**的任务天然可并行
- **不同专业领域**的任务分配给不同 Agent 时自动并行
- 示例：`[论文翻译]` 和 `[图表绘制]` 可并行，但都依赖 `[数据整理]`

### 任务大小估算

| 级别 | 预估时间 | 示例 |
|------|----------|------|
| S | <10min | 修复一个 bug、写一个工具函数 |
| M | 10-30min | 实现一个 API 端点、写一个测试模块 |
| L | 30-60min | 完整功能模块、复杂重构 |

## 2. Agent 分配策略

### 按专长分配
将任务匹配到最擅长的 Agent，而非最空闲的 Agent：

```
任务：实现 Transformer 模型 → 分配给 scholar（学术研究）
任务：写部署脚本 → 分配给 auto（自动化）
任务：分析市场数据 → 分配给 stock（股票分析）
```

### 复杂度与模型匹配
- **简单重复**（数据提取、格式转换）→ 轻量模型
- **创意生成**（文案、方案设计）→ 强模型
- **代码实现**（中等复杂度）→ 中等模型
- **架构决策、多步推理**→ 最强模型

### 跨 Agent 通信时机
- **必要通信**：任务有显式依赖、需要上下文传递
- **避免不必要通信**：独立任务不要互相等待
- **批量传递**：多个数据点打包一次发送，而非逐条通信

## 3. 质量门禁实施

### 验收标准编写
每个任务应包含可验证的完成条件：

```
任务：实现登录 API
验收标准：
  1. POST /auth/login 返回 JWT token
  2. 密码错误返回 401
  3. 单元测试覆盖率 > 80%
  4. 通过集成测试
```

### 结果验证清单
- [ ] 输出物是否存在且格式正确？
- [ ] 是否满足所有验收标准？
- [ ] 是否引入了新问题（回归）？
- [ ] 文档/注释是否更新？

### 失败重试策略
- **可恢复错误**（网络超时、API 限流）→ 自动重试，最多 3 次，指数退避
- **逻辑错误**（代码 bug、方案错误）→ 返回上游 Agent 修正，记录错误模式
- **需求不明确**→ 暂停并请求人工澄清，不要猜测

## 4. 进度管理

### 状态报告格式
```
📊 任务进度 [2026-03-31 17:00]
━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ T1 数据采集 (auto) - 已完成
🔄 T2 数据清洗 (auto) - 进行中 (60%)
⏳ T3 模型训练 (scholar) - 等待 T2
⏳ T4 结果分析 (scholar) - 等待 T3
━━━━━━━━━━━━━━━━━━━━━━━━━━
风险：T2 数据质量可能影响 T3 训练效果
```

### 阻塞任务处理
1. 识别阻塞源（哪个任务卡住了？）
2. 评估替代方案（能否跳过？用占位数据？）
3. 通知依赖方并调整计划
4. 记录阻塞原因以便复盘

### 计划修订原则
- 小幅调整（任务顺序、并行度）→ 直接修改，通知相关 Agent
- 大幅调整（目标变更、新增需求）→ 重新分解，生成新计划
- 始终保留修订记录，便于追溯

## 5. 借鉴 Claude Code Agent Teams 的经验

### 共享任务列表模式
所有 Agent 读写同一份任务文件（如 `TASK_BOARD.md`），通过文件锁或状态标记避免冲突。适合 2-4 个 Agent 协作的小型项目。

### Lead + Teammates 架构
- **Lead**：负责任务分解、分配、验收，不直接执行
- **Teammates**：专注执行分配到的任务，完成后汇报
- OpenClaw 的 `leader` Agent 天然适合 Lead 角色

### 通信模式选择

| 模式 | 适用场景 | 优缺点 |
|------|----------|--------|
| Hub-and-spoke | Leader 统一协调 | ✅ 简单可控 ❌ Leader 成为瓶颈 |
| Mesh | Agent 间直接通信 | ✅ 低延迟 ❌ 复杂、难追踪 |
| 混合 | Lead 协调 + 专业 Agent 直连 | ✅ 平衡 ✅ 推荐 |

### Subagents vs Agent Teams
- **Subagents**（当前模式）：适合单次任务、短生命周期、Leader 直接监督
- **Agent Teams**：适合长期项目、需要 Agent 间深度协作、任务边界模糊时
- 经验法则：任务可在 1 小时内完成 → Subagents；需要多轮迭代 → Agent Teams

## 6. 通信架构选择：Mesh vs Hub-and-spoke

### 模式对比

| 维度 | Hub-and-spoke | Mesh |
|------|---------------|------|
| 路径 | 所有消息经 orchestrator 中转 | Agent 间直接通信 |
| 适合 | 强依赖链、成本敏感、需统一决策 | 辩论/质疑、探索性研究、跨领域协调 |

### 决策流程图（文字描述）

```
1. 任务间是否有强依赖？ → 是 → Hub-and-spoke
2. Agent 是否需要辩论或质疑彼此结论？ → 是 → Mesh
3. 是否涉及关键决策点（方案选型、架构设计）？ → 是 → Mesh
4. 成本是否高度敏感？ → 是 → Hub-and-spoke
5. 默认 → Hub-and-spoke
```

### 混合模式（推荐）

日常用 Hub-and-spoke 保持可控，遇到关键决策点临时切换为 Mesh。示例：论文写作流程中，数据采集和文献综述用 Hub-and-spoke，但确定实验方案时让 scholar 和 auto 直接讨论可行性。

## 7. 能力分类与 Agent 映射

| 能力类型 | 描述 | 适合的 Agent | 搜索关键词 |
|----------|------|-------------|-----------|
| browser_automation | 网页操作、截图、表单填写 | auto | agent-browser, playwright |
| web_search | 信息检索、新闻查找 | stock, scholar | tavily, exa-search |
| api_integration | 第三方 API 调用与对接 | auto | api, rest, webhook |
| data_extraction | 从网页/文件/PDF 提取结构化数据 | auto, scholar | scraping, parser |
| data_transformation | 数据清洗、格式转换、聚合 | auto, stock | transform, etl |
| content_generation | 文案、报告、代码生成 | creator, scholar | writing, generation |
| file_operations | 读写、编辑、压缩文件 | auto | file, fs |
| message_delivery | 发送通知到 Telegram/飞书等 | leader | telegram, notify |
| scheduling | 定时任务、cron 管理 | auto, leader | cron, scheduler |
| authentication | 登录、token、OAuth | auto | auth, login |
| database_operations | SQL/NoSQL 读写、迁移 | auto | database, sql |
| code_execution | 运行脚本、编译、构建 | auto | exec, runner |
| version_control | Git 操作、PR 管理 | auto | git, version-control |
| testing | 单元/集成/E2E 测试 | auto | testing, jest |
| deployment | CI/CD、容器、服务器部署 | auto | deploy, docker |
| monitoring | 日志、指标、告警 | auto, stock | monitor, logging |

## 8. 任务认领 vs 分配的决策树

```
1. 任务是否需要特定专长？ → 是 → orchestrator 分配
2. 任务是否需要严格的执行顺序？ → 是 → orchestrator 分配
3. Agent 是否有足够的上下文判断该做什么？ → 是 → 允许认领
4. 任务是否为探索性/开放性？ → 是 → 允许认领（激发主动性）
5. 默认 → orchestrator 分配
```

**示例**：重构登录模块 → 分配（需要 auto 的代码专长）；调研新技术方向 → 允许 scholar 自主认领（探索性任务）。

## 9. 成本优化策略

| 复杂度 | 模式 | 成本特征 | 示例 |
|--------|------|----------|------|
| 低 | 直接执行 | 零额外成本 | 修改配置文件、查天气 |
| 中 | Subagents | Σ 各 subagent token | 写一个脚本 + 并行测试 |
| 高 | Agent Teams | 各 agent 独立上下文 × 数量 | 多人协作开发完整项目 |

**优化建议**：能用 subagent 就不用 agent teams；并发数控制在 3 以内；完成的 session 及时清理，避免上下文膨胀。

## 10. 常见反模式与解决方案

| 反模式 | 症状 | 解决方案 |
|--------|------|----------|
| 过度分解 | 任务数 > 20，管理开销 > 执行开销 | 合并同类任务，控制在 5-15 个 |
| 串行瀑布 | 所有任务排队执行，无并行 | 识别无依赖任务，强制并行 |
| 静默失败 | Agent 完成但结果不符合预期 | 每个任务必须有验收标准 |
| Leader 瓶颈 | 所有决策都等 Leader 确认 | 下放决策权，仅关键节点需确认 |
| 通信风暴 | Agent 间频繁发消息 | 批量通信，减少不必要交互 |
| 计划僵化 | 环境变化但计划不调整 | 定期检查，允许动态调整 |
| 责任模糊 | 多个 Agent 都以为别人在做 | 每个任务明确唯一负责人 |
| 忽视风险 | 计划只列任务不列风险 | 任务分解时同步识别风险点 |
