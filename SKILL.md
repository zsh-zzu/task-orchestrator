---
name: task-orchestrator
description: Complex task decomposition and multi-agent orchestration with shared task lists, inter-agent communication, quality gates, and learning. Use when: (1) user gives a complex task requiring multiple steps or agents, (2) user asks to break down/decompose/split a task, (3) user says "orchestrate", "复杂任务", "拆分任务", "子任务", (4) managing multi-phase projects with parallel workstreams, (5) coordinating specialized agents on a shared project. NOT for: simple one-shot requests, single tool calls, conversational exchanges, or tasks with tight sequential dependencies.
---

# Task Orchestrator

## Decision: Simple or Complex?

### Simple → handle directly. ALL true:
- Single clear action, one agent can complete, no dependencies

### Complex → proceed below. ANY true:
- 3+ steps with dependencies, spans multiple agents/tools, multi-phase/parallel workstreams

### Complexity Routing

| Complexity | Strategy | When |
|-----------|----------|------|
| Low (1-2 steps) | Direct execution | Simple queries, single tool calls |
| Medium (3-6 steps) | Subagents | Independent parallel tasks, result-only needed |
| High (7+ steps or cross-domain) | Agent Teams | Inter-agent discussion, shared context, collaboration |

> ⚠️ **Cost awareness:** Agent Teams orchestration has significantly higher token cost than subagents. Only use Agent Teams when true inter-agent discussion/collaboration is needed. For simple parallel tasks, prefer subagents.

## Phase 0: Clarify (Before Anything Else)

**CRITICAL: When the request is ambiguous, incomplete, or has multiple valid interpretations, STOP and ask the user BEFORE decomposing or dispatching.**

### When to Ask

- Goal is vague: "帮我做一个网站" → 什么类型？面向谁？核心功能？
- Missing constraints: 技术栈、预算、截止时间、部署环境未指定
- Conflicting info: 任务 A 和任务 B 似乎矛盾
- Multiple valid approaches: 没有明确偏好时，列出 2-3 个方案让用户选
- Assumptions needed: 任何"我猜用户想要..."的想法，都应该变成确认问题
- Unclear scope: 不确定哪些部分在范围内、哪些不在
- Dependencies unclear: 不确定某个前置条件是否已满足（如账号、权限、数据）

### How to Ask

- **一次问完**：收集所有不确定点，一次性列出，避免来回多次打扰用户
- **提供选项**：不要只问"你想怎么做"，给具体选项让用户选（A/B/C）
- **说明影响**：解释不同选择会导致什么不同的执行路径或结果
- **带默认建议**：如果有一个明显更好的方案，推荐它但让用户最终决定
- **标注优先级**：如果有些问题不影响整体推进，标记为"可后补"，先推进确定的部分

### Format

```
🤔 开始分解任务前，有几个地方需要确认：

**必须确认：**
1. **[问题1]** — [为什么需要确认] → A / B / C？
2. **[问题2]** — [为什么需要确认] → [选项]

**可后补：**
3. **[问题3]** — [为什么影响不大]

我的建议是：[推荐方案]，理由是 [简短理由]。
```

### When NOT to Ask

- 请求已经很明确，没有歧义
- 用户说"你自己决定"或"按你的判断"
- 问题太小，任何选择都不影响结果
- 紧急任务，延迟确认的代价 > 做错选择的代价（先做，在 tracking file 中标注假设，后续可调整）
- 已有明确上下文（如之前讨论过类似任务，用户引用了之前的结论）

### Clarification During Execution

Clarify 不只在开始前。执行过程中如果遇到新问题：
- 子 agent 反馈需求不明确 → orchestrator 向用户确认后转发
- 发现依赖缺失（如需要某个 API key、账号权限）→ 立即向用户报告，不要静默跳过
- 技术方案遇到死胡同 → 向用户说明情况，提供替代方案

**原则：宁可多问一次，也不要猜错了重做。**

---

## Phase 0.5: Capability Check (Match Skills to Needs)

**在正式分解任务之前，先盘点可用的 skills 和工具，确保每个子任务都有能力支撑。**

### Steps

1. **从 Clarify 结果提取能力需求** — 根据已确认的需求，列出需要什么能力（搜索、浏览器、文件操作、代码生成、图片处理等）
2. **扫描可用 skills** — 检查当前工作区 `skills/` 目录下已安装的 skills
3. **检查内置工具** — 评估内置工具是否能满足需求（exec、browser、web_fetch、pdf、image 等）
4. **识别能力缺口** — 对比需求 vs 可用能力，标记缺失的部分

### Capability Map Format

在 tracking file 中记录：

```markdown
## Capabilities
| 需求 | 可用 Skill/Tool | 状态 |
|------|----------------|------|
| 网络搜索 | multi-search-engine | ✅ 就绪 |
| 浏览器操作 | agent-browser | ✅ 就绪 |
| PDF 分析 | 内置 pdf tool | ✅ 就绪 |
| 视频生成 | — | ❌ 缺失 |
| 数据可视化 | — | ⚠️ 需确认 |
```

### Gap Handling

发现能力缺口时，按优先级处理：

| 优先级 | 情况 | 处理方式 |
|--------|------|---------|
| P0 | 缺核心能力，任务无法执行 | **立即告知用户**，说明缺什么，建议安装 skill 或换方案 |
| P1 | 有替代方案但效果差 | 告知用户替代方案及其局限，让用户决定 |
| P2 | 缺辅助能力但不阻塞 | 标注在 tracking file 中，执行时灵活处理 |

### If No Skill Exists

When you need a capability but no skill exists:

**Step 1: Search ClawHub**
```
clawhub search <capability>
```
- If found → propose installation, wait for user confirmation
- `clawhub install <skill-name>`

**Step 2: Check if built-in tools can substitute**
- Web search → `web_search` or `multi-search-engine` skill
- Browser → `browser` tool with `profile="user"`
- File ops → `read`/`write`/`edit` tools
- Code exec → `exec` tool

**Step 3: Create new skill (if needed)**
If no existing solution and the need is recurring:
1. Use `skill-creator` skill to scaffold a new one
2. Define: name, description, trigger words, workflow steps
3. Test on current task, refine, then install globally

**Step 4: Adjust scope**
If skill creation is overkill or user doesn't want to wait:
- Reduce task scope to what's currently possible
- Mark the stretch goal as "future enhancement"
- Don't silently skip functionality

### Skill Gap Decision Tree

```
Need capability X
    ↓
Search clawhub → Found? → Install ✓
    ↓ No
Built-in tool works? → Use it ✓
    ↓ No
Recurring need? → Create skill ✓
    ↓ No
Adjust task scope → Inform user
```

### Output

这一步的输出直接输入到 Phase 1 Decompose：
- **已确认可用的能力** → 放心分配给对应子任务
- **能力缺口** → 要么先解决（安装 skill），要么调整任务设计绕过缺口

---

## Phase 1: Decompose

### Role Templates

Standard agent roles for multi-agent teams. Every agent has ONE primary role — overlap causes confusion.

| Role | Purpose | Model guidance |
|------|---------|---------------|
| **Orchestrator** | Route work, track state, make priority calls | High-reasoning model (handles judgment) |
| **Builder** | Produce artifacts — code, docs, configs | Can use cost-effective models for mechanical work |
| **Reviewer** | Verify quality, push back on gaps | High-reasoning model (catches what builders miss) |
| **Ops** | Cron jobs, standups, health checks, dispatching | Cheapest reliable model |

### Task States

Every task moves through a defined lifecycle:

```
Inbox → Assigned → In Progress → Review → Done | Failed
```

**Rules:**
- Orchestrator owns state transitions — don't rely on agents to update their own status
- Every transition gets a comment (who, what, why)
- Failed is a valid end state — capture why and move on

### Rules
- Read the FULL request before decomposing
- Each subtask: clear deliverable + acceptance criteria + assigned agent
- Action-oriented names: "Build X", "Analyze Y", "Deploy Z"
- Mark real dependencies only — parallel what CAN be parallel
- Size: `(S)` <30min, `(M)` 30min-2h, `(L)` 2h+

### Tracking File

Create `workspace/tasks/{slug}-{YYYY-MM-DD-HHmm}.md`:

```markdown
# Plan: {task name}
Started: {date} | Status: in-progress | Revised: 0

## Tasks
- [ ] 1. (S) task description — agentId
      ↳ parallel with 2
- [ ] 2. (M) task description (depends: 1) — agentId
- [ ] 3. (L) task description (depends: 2) — agentId
      ⚠️ blocked: reason

## Retro
<!-- filled on completion -->
```

**Notation:** `↳ parallel with N` | `(depends: N)` | `⚠️ blocked: reason` | `spawned: session-id`
Done: `- [x]` with result summary after dash.

### Revision
- Insert: next number, update downstream `depends:` | Remove: ~~strikethrough~~, don't renumber
- Bump `Revised:` counter | Don't follow a bad plan to completion

## Phase 2: Dispatch

### Dispatch Modes

| Mode | How | When to use |
|------|-----|-------------|
| **Lead Assign** (default) | Orchestrator assigns tasks to agents | Task boundaries are clear, requirements well-defined |
| **Self-claim** | Agents pick tasks from shared task list | Exploratory tasks, autonomous teams, unclear scope |

**Decision rule:** Clear deliverables → Lead Assign. Exploratory/creative work → Self-claim.

### Agent Assignment
Map by AGENTS.md role definitions. Fallback: use `runtime="subagent"` with default agent.

### Spawn Pattern
```
sessions_spawn(agentId=<agent>, task=<detailed subtask + context + acceptance criteria + output path>, mode="run", label="task-{slug}-{number}")
```

### Batch Rules
- Respect `maxConcurrent` (default 8), batch 2-3 at a time, 10-15s between batches
- Sequential if B depends on A's output | One subagent per independent branch

## Phase 3: Monitor & Review

### Progress Check
Use `subagents(action="list")` — don't poll in loops, check on events or when asked.

### Status Report Format
```
📋 Plan: {name} | Progress: 3/7 tasks (1L + 2S remaining) | Current: Task 4 (L) | Blocked: Task 5 — waiting on X
```

### Quality Gates

Every completed subtask goes through quality check:
1. **Acceptance criteria met?** → compare result against stated criteria
2. **Output exists at expected path?** → verify artifacts
3. **No regressions?** → downstream tasks still viable

- ✅ Pass → update tracking file | ❌ Fail → retry once, then escalate | ⚠️ Partial → fix or flag

**Delivery Proof:** If a task claims external delivery (message sent, post published, etc.), require: target channel ID, message ID, and verifiable location description. Verbal confirmation alone does NOT count as delivery complete.

### Inter-Agent Communication

For medium+ complexity, enable direct agent-to-agent messaging:
```
sessions_send(agentId=<target>, message=<context + findings + question>)
```

Use when: handoff with context, debate/challenge findings, cross-domain coordination.

### Handoff Protocol

Structured交接 messages between agents must include:
1. **What was done** — completed work summary
2. **Where artifacts are** — file paths, URLs, references
3. **How to verify** — acceptance criteria or test commands
4. **Known issues** — caveats, incomplete areas, risks
5. **What's next** — recommended follow-up or blocking decisions

### Anti-drop Guard

After each subagent completes, orchestrator MUST verify:
1. The task exists on the task list
2. The task is not yet marked done (update it now)
3. The next task has been dispatched OR explicitly marked blocked/idle

If any check fails, the current turn is NOT complete. No tasks silently fall through the cracks.

### Blocked Tasks
Mark it, work next unblocked task. Don't let dependent lines drift apart. Auto-resume when blocker clears.

### Failure Handling
Task fails → note it, assess downstream viability. Unrecoverable → `abandoned`, explain why. Record partial progress.

## Phase 4: Complete & Learn

### Completion Checklist
- [ ] All subtasks done or abandoned | [ ] Tracking file → `completed` | [ ] Results synthesized | [ ] Artifacts verified | [ ] Summary delivered

### Retro (Learning Loop)

When a plan finishes (completed or abandoned), fill the `## Retro` section:

```markdown
## Retro
- **What worked:** [patterns that helped] | **What didn't:** [failures/inefficiencies]
- **Sizing accuracy:** X/N correct → which tasks were mis-sized?
- **Dependencies missed:** Any surprise blockers that weren't on the plan?
- **Reusable pattern:** [If this plan type recurs, note the template]
```

### Patterns File (`workspace/tasks/patterns.md`)

Distill lessons from retros into reusable patterns:

```markdown
# Planning Patterns (learned from retros)

## Sizing Rules
- API integration tasks: usually (M), not (S)
- "Write tests" is always bigger than you think → size up one level
- Database migrations with data backfill: always (L)

## Dependency Rules
- Auth must come before any authenticated endpoint work
- Don't parallelize tasks that write to the same config file

## Templates
- **API build:** provision → auth + endpoints (parallel) → tests → deploy
- **Research report:** gather sources → analyze (parallel per source) → synthesize → review
```

This file gets checked during Planning. Over time, sizing gets more accurate, dependency mistakes stop repeating, and common project types get reusable templates.

### Pattern Decay

Review `patterns.md` monthly. Remove patterns that:
- Haven't applied in 30+ days
- Were wrong more than once
- Are too project-specific to generalize

**Skipping retros = same mistakes forever.** Every completed plan must have a retro.

### Lifecycle & Cleanup

Plans follow a strict lifecycle to prevent accumulation:

```
active/ → completed → archive/ → deleted (after pattern extraction)
```

**On completion:**
1. Fill Retro, extract reusable patterns to `patterns.md`
2. Move tracking file: `workspace/tasks/` → `workspace/tasks/archive/`
3. Prefix with date: `{date}-{slug}.plan.md`

**Retention policy:**
- Keep max 20 archived plans in `archive/`
- Keep max 30 days (delete older)
- Abandoned plans: archive immediately, no pattern extraction needed

**Cleanup routine (run before each new orchestration):**
1. Count files in `archive/`
2. If > 20: delete oldest files beyond limit
3. Check dates: delete files older than 30 days
4. Check `patterns.md` for stale entries (unused 30+ days) and remove them

**Pattern extraction then delete:**
- After extracting reusable patterns from an archived plan into `patterns.md`, the archived plan can be safely deleted — the knowledge is preserved in patterns
- Only keep archived plans that have NOT been fully extracted yet

## Advanced Collaboration Patterns

### Competitive Hypotheses
Multiple agents test different theories in parallel, challenge each other, converge on best solution.
**When:** Research questions, algorithm selection, root cause analysis.
**Example:** Agent A tests "memory leak in cache" while Agent B tests "connection pool exhaustion" — they share findings and converge.

### Debate & Consensus
Agents hold different positions and argue architecture/design decisions.
**When:** Major architectural choices, tech stack selection, design trade-offs.
**Example:** Agent advocates microservices, Agent advocates monolith — structured debate with criteria scoring.

### Layered Orchestration
Phase 1 (Planning): define roles, boundaries, interfaces → Phase 2 (Execution): parallel build.
**When:** Large features with many moving parts, cross-cutting concerns.
**Example:** Planning phase defines API contracts → parallel frontend/backend/spawn build to those contracts.

## Anti-patterns

- **Over-decomposition:** 15 tasks for 4 steps of real work
- **Phantom dependencies:** Marking sequential what's actually parallel
- **Plan worship:** Following a wrong plan because it exists
- **Poll loops:** Repeatedly calling subagents list
- **Orchestrator doing execution:** Route and track, don't build yourself
- **Communication bottleneck:** Forcing all chat through orchestrator when direct messaging works
- **Ignoring partial results:** Failed tasks may still produce useful artifacts
- **Unverified delivery:** Accepting "done" without proof of external delivery
