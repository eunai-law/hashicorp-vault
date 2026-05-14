# /gsd-progress

## Purpose
Shows the current project position and recommends the next step. The `--next` flag auto-detects and runs the next command automatically.

## When to Use
- At the start of a session to re-orient after a break
- When unsure what to run next in the GSD loop
- As the entry point for autonomous mode

## Context Consumed
- `.planning/STATE.md` — current phase, plan, status
- `.planning/ROADMAP.md` — phase completion status

## Artifacts Created
None (read-only).

## Agents Spawned
None.

## Token Cost Profile
**Very Low** — reads STATE.md and ROADMAP.md, displays summary. Under 2K tokens.

## Flags
```bash
/gsd-progress         # show current position
/gsd-progress --next  # auto-detect and run next step
```

## Output Example
```
Current Position: Phase 2 (user-auth) — Execute
Plans: 3/4 complete (plan-4 pending)
Next recommended: /gsd-execute-phase 2 --plan 4

Phase Progress:
Phase 1 ████████████ Complete
Phase 2 █████████░░░ 75% (executing)
Phase 3 ░░░░░░░░░░░░ Pending
```
