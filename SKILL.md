---
name: task-orchestrator
description: Complex task decomposition and multi-agent orchestration with shared task lists, inter-agent communication, quality gates, and learning. Use when: (1) user gives a complex task requiring multiple steps or agents, (2) user asks to break down/decompose/split a task, (3) user says "orchestrate", "复杂任务", "拆分任务", "子任务", (4) managing multi-phase projects with parallel workstreams, (5) coordinating specialized agents on a shared project. NOT for: simple one-shot requests, single tool calls, conversational exchanges, or tasks with tight sequential dependencies.
---

# Task Orchestrator

## Decision: Simple or Complex?

### Simple → handle directly. ALL true:
- Single clear action
- One agent can complete
- No dependencies between steps

### Complex → proceed below. ANY true:
- 3+ steps with dependencies
- Spans multiple agents/tools
- Multi-phase or parallel workstreams
- Requires inter-agent coordination

### Complexity Routing

| Complexity | Strategy | When |
|-----------|----------|------|
| Low (1-2 steps) | Direct execution | Simple queries, single tool calls |
| Medium (3-6 steps) | Subagents | Independent parallel tasks, result-only needed |
| High (7+ steps or cross-domain) | Agent Teams | Need inter-agent discussion, shared context, collaboration |

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

**Notation:**
- `↳ parallel with N` = concurrent
- `(depends: N)` = must complete first
- `⚠️ blocked: reason`
- `spawned: session-id`
- Done: `- [x]` with result after dash: `— result summary`

### Revision
- Insert: next number, update downstream `depends:`
- Remove: ~~strikethrough~~, don't renumber
- Bump `Revised:` counter
- Don't follow a bad plan to completion

## Phase 2: Dispatch

### Agent Assignment
Map by AGENTS.md role definitions. Fallback: use `runtime="subagent"` with default agent.

### Spawn Pattern
```
sessions_spawn(
  agentId=<agent>,
  task=<detailed subtask with context + acceptance criteria + output path>,
  mode="run",
  label="task-{slug}-{number}"
)
```

### Batch Rules
- Respect `maxConcurrent` (default 8)
- Batch 2-3 at a time, 10-15s between batches
- Sequential if subtask B depends on A's output
- One subagent per independent branch, not per task

## Phase 3: Monitor & Review

### Progress Check
Use `subagents(action="list")` — don't poll in loops, check on events or when asked.

### Status Report Format (size-aware)
```
📋 Plan: {name}
Progress: 3/7 tasks (1L + 2S remaining)
Current: Task 4 (L) — running
Blocked: Task 5 — waiting on X
Revised: 1x
```

### Quality Gates

Every completed subtask goes through quality check:
1. **Acceptance criteria met?** → compare result against stated criteria
2. **Output exists at expected path?** → verify artifacts
3. **No regressions?** → downstream tasks still viable

- ✅ Pass → update tracking file, mark done
- ❌ Fail → retry once with refined instructions, then escalate to user
- ⚠️ Partial → note gaps, decide: fix inline or flag for user

### Inter-Agent Communication

For medium+ complexity, enable direct agent-to-agent messaging:
```
sessions_send(agentId=<target>, message=<context + findings + question>)
```

Use when:
- Agent A's output is input for Agent B (handoff with context)
- Agents need to debate/challenge findings
- Cross-domain coordination needed (e.g., frontend ↔ backend)

### Blocked Tasks
- Mark it, work next unblocked task
- Don't let dependent lines drift apart
- Auto-resume when blocker clears

### Failure Handling
- Task fails → note it, assess: can downstream still run?
- Unrecoverable → Status: `abandoned`, explain why
- Record what DID work — partial progress has value

## Phase 4: Complete & Learn

### Completion Checklist
- [ ] All subtasks marked done or explicitly abandoned
- [ ] Tracking file status updated to `completed`
- [ ] Results synthesized into coherent summary
- [ ] Artifacts collected and paths verified
- [ ] Summary delivered to user

### Retro (Learning Loop)

Fill `## Retro` section in tracking file:
```markdown
## Retro
- **What worked:** [specific patterns that helped]
- **What didn't:** [specific failures or inefficiencies]
- **Sizing accuracy:** X/N tasks sized correctly
- **Dependencies missed:** [if any]
- **Reusable pattern:** [template for similar future tasks]
```

### Patterns File (`workspace/tasks/patterns.md`)
Distill lessons for future planning:
```markdown
# Planning Patterns

## Sizing
- [learned sizing rules]

## Dependencies
- [learned dependency rules]

## Templates
- **API build:** provision → auth + endpoints (parallel) → tests → deploy
```

### Archive
Completed plans → rename `{date}-{slug}.plan.md` in `workspace/tasks/archive/`

## Anti-patterns

- **Over-decomposition:** 15 tasks for 4 steps of real work
- **Phantom dependencies:** Marking sequential what's actually parallel
- **Plan worship:** Following a wrong plan because it exists
- **Size blindness:** "4/5 done!" when remaining task is (L)
- **No review:** Accepting results without checking acceptance criteria
- **Poll loops:** Don't repeatedly call subagents list; check on events
- **Orchestrator doing execution:** Route and track, don't build yourself
- **Communication bottleneck:** Forcing all inter-agent chat through orchestrator when direct messaging is available
- **Ignoring partial results:** A failed task may still produce useful artifacts for downstream work
