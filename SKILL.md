---
name: task-orchestrator
description: Complex task decomposition and subagent orchestration with monitoring, review, and learning. Use when: (1) user gives a complex task requiring multiple steps or agents, (2) user asks to break down/decompose/split a task, (3) user says "orchestrate", "复杂任务", "拆分任务", "子任务", (4) managing multi-phase projects with parallel workstreams. NOT for: simple one-shot requests, single tool calls, or conversational exchanges.
---

# Task Orchestrator

## Decision: Simple or Complex?

**Simple** → handle directly. ALL true:
- Single clear action
- One agent can complete
- No dependencies between steps

**Complex** → proceed below. ANY true:
- 3+ steps with dependencies
- Spans multiple agents/tools
- Multi-phase or parallel workstreams

## Phase 1: Decompose

### Rules
- Read the FULL request before decomposing
- Each subtask: clear deliverable + acceptance criteria + one agent
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
Map by AGENTS.md. Fallback: use `runtime="subagent"` with default agent.

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
Use `subagents(action="list")` — don't poll in loops, check on events.

### Status Report Format
```
📋 Plan: {name}
Progress: 3/7 tasks (1L + 2S remaining)
Current: Task 4 (L) — running
Blocked: Task 5 — waiting on X
Revised: 1x
```

Size-aware > task-count.

### Review on Completion
When a subagent completes:
1. Check result against acceptance criteria
2. ✅ Pass → update tracking file, mark done
3. ❌ Fail → retry once with refined instructions, then report to user
4. Record outcome in tracking file

### Blocked Tasks
- Mark it, work next unblocked task
- Don't let dependent lines drift apart

### Failure Handling
- Task fails → note it, assess: can downstream still run?
- Unrecoverable → Status: `abandoned`, explain why
- Record what DID work — partial progress has value

## Phase 4: Complete

When all subtasks done:
1. Update tracking file status to `completed`
2. Fill `## Retro` section:
   - What worked / What didn't
   - Sizing accuracy
   - Dependencies missed
   - Reusable patterns
3. Archive: rename to `{date}-{slug}.plan.md`
4. Synthesize results → coherent summary to user
5. Update `workspace/tasks/patterns.md` with learned lessons

### Patterns File (`patterns.md`)
```markdown
# Planning Patterns
## Sizing
- [learned sizing rules]

## Dependencies
- [learned dependency rules]

## Templates
- **API build:** provision → auth + endpoints (parallel) → tests → deploy
```

## Anti-patterns

- **Over-decomposition:** 15 tasks for 4 steps of real work
- **Phantom dependencies:** Marking sequential what's actually parallel
- **Plan worship:** Following a wrong plan because it exists
- **Size blindness:** "4/5 done!" when remaining task is (L)
- **No review:** Accepting results without checking acceptance criteria
- **Poll loops:** Don't repeatedly call subagents list; check on events
- **Orchestrator doing execution:** Route and track, don't build yourself
