# gsd-executor

## Role
Implements PLAN.md files atomically. Reads a plan, implements each task sequentially, creates one git commit per task, handles deviations automatically, and produces a SUMMARY.md. The primary code-writing agent in GSD.

## Inputs
- `PLAN.md` — the plan to execute (tasks, must-haves, key links, verification commands)
- `STATE.md` — current project state and decisions
- `PROJECT.md` — project vision and constraints
- Prior wave `SUMMARY.md` files (for 500K+ context windows — adaptive enrichment)
- Relevant codebase files (read during implementation)

## Outputs
- Code files (implementation)
- `SUMMARY.md` — per-plan execution summary (tasks completed, files modified, decisions made, deviations from plan)
- `STATE.md` updates (via lockfile-protected writes)
- Git commits (one per task, atomic)

## Lifecycle
1. Receives orchestrator payload (PLAN.md content + context JSON)
2. Reads codebase files relevant to plan tasks
3. Implements Task 1 → stages changes → `git commit -m "feat(phase-N): [task description]"`
4. Implements Task 2 → commit
5. Implements Task 3 → commit
6. Runs verification commands from PLAN.md
7. Writes SUMMARY.md
8. Updates STATE.md via `gsd-sdk query state update-progress`

## Orchestration Trigger
Spawned by `execute-phase` orchestrator, one executor per PLAN.md file. Executors within the same wave spawn simultaneously.

## Isolation
**Fresh 200K-token context.** Does not inherit the orchestrating session's conversation history. Receives only its structured payload.

**Tool access:** Read, Write, Edit, Bash, Grep, Glob, mcp__context7__*

## Token Behavior
**High** — reads codebase files, implements code, runs tests. Typical range: 25K–80K tokens per executor depending on plan complexity and codebase size.

Context7 MCP is available for library documentation lookups. If Context7 is unavailable, the executor falls back to `ctx7` CLI tool before giving up.

## Key Behaviors

### Deviation Handling
If the executor encounters a situation not covered by the plan (e.g., a file doesn't exist, an API has changed), it:
1. Makes the minimum necessary adaptation
2. Documents the deviation in SUMMARY.md
3. Does not stop execution — continues to next task

Deviations that would require architectural decisions are escalated to the orchestrator via a checkpoint signal.

### Checkpoint Protocol
For blocking decisions (cannot proceed without human input), the executor writes a `state signal-waiting` entry and pauses. The orchestrator detects this and surfaces it to the user.

### Git Commit Format
```
feat(phase-{N}): {task description in present tense}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

### --no-verify Usage
Within parallel waves, commits use `--no-verify` to avoid build-lock contention. The orchestrator runs hooks after each wave completes. Sequential execution uses normal commits with hooks.

### SUMMARY.md Content
```markdown
---
phase: 1
plan: 2
status: complete
tasks_completed: 3
files_modified: 7
---

## What Was Done
[Task-by-task narrative]

## Decisions Made
[Any choices made during implementation]

## Deviations from Plan
[Anything that differed from the PLAN.md spec]

## Verification
[Output of verification commands run]
```
