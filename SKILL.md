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

## Phase 1: Decompose

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

Fill `## Retro` in tracking file:
```markdown
## Retro
- **What worked:** [patterns that helped] | **What didn't:** [failures/inefficiencies]
- **Sizing accuracy:** X/N correct | **Reusable pattern:** [template for future]
```

### Patterns File (`workspace/tasks/patterns.md`)
Distill lessons: sizing rules, dependency rules, templates (e.g., "API build: provision → auth+endpoints → tests → deploy").

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
