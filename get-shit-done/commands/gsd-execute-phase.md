# /gsd-execute-phase N

## Purpose
Executes all PLAN.md files for a phase using parallel wave-based execution. Each plan runs in its own fresh 200K-token executor agent. Each task within a plan becomes an atomic git commit. Your main context window stays at 30–40%.

## When to Use
- After `/gsd-plan-phase` has produced approved PLAN.md files
- To re-run failed plans after `/gsd-verify-work` identifies gaps
- With `--plan M` to re-run a single plan

## Context Consumed
- All `.planning/phases/N/plan-{M}.md` files (wave grouping and dependency info)
- `.planning/STATE.md` — current phase state
- `.planning/PROJECT.md` — project constraints
- Phase context: prior wave SUMMARY.md files (for cross-plan awareness in 500K+ context windows)

## Artifacts Created
| File | Contents |
|---|---|
| Code files | Implementation (language-specific) |
| `.planning/phases/N/summary-{M}.md` | Per-plan execution summary (tasks done, files changed, decisions made) |
| `.planning/STATE.md` | Updated progress, metrics, decisions |
| Git commits | One per task, atomic, with semantic messages |

## Agents Spawned
One `gsd-executor` per PLAN.md file, grouped into dependency waves:

```
Wave 1: [plan-1, plan-2, plan-3]  → all spawn simultaneously
Wave 2: [plan-4]                  → spawns after all Wave 1 complete
```

## What Each Executor Does
1. Reads its PLAN.md
2. Reads relevant codebase files
3. Implements Task 1 → `git commit` (atomic)
4. Implements Task 2 → `git commit` (atomic)
5. Implements Task 3 → `git commit` (atomic)
6. Writes SUMMARY.md
7. Updates STATE.md (lockfile-protected)

## Context Refresh
Yes — each executor starts with a fresh 200K context. This is the primary mechanism that keeps execution quality consistent regardless of session age.

Within a wave, executors use `git commit --no-verify` to avoid build-lock contention. The orchestrator runs hooks once after each wave.

## Token Cost Profile
**Very High**

Linear in: (number of plans) × (plan depth) × (codebase context per executor)

Estimates:
- Simple phase (3 plans × 3 tasks each): ~3 × 30K = ~90K tokens
- Medium phase (5 plans × 3 tasks each): ~5 × 40K = ~200K tokens
- Complex phase (8 plans × 4 tasks each): ~8 × 50K = ~400K tokens

This is typically the highest per-command cost in the GSD workflow. It scales directly with phase complexity.

**Cost reduction:**
- Use `budget` model profile for execution (Haiku is fast and sufficient for mechanical implementation)
- Keep plans small (2–3 tasks per plan) to reduce per-executor context depth
- Disable parallelization for small phases (reduces concurrent context initialization overhead)

## Common Pitfalls
- **Running execute before plan-checker approves**: Executors receive unapproved plans and make bad implementation decisions. Always let plan-phase complete.
- **No git clean state before execution**: Uncommitted changes can conflict with executor commits. Commit or stash everything before running.
- **Parallelization with shared config files**: Two executors both modifying `config.ts` or `package.json` simultaneously create merge conflicts. Use dependency waves to serialize shared-file modifications.
- **Very large plans**: Plans with 6+ tasks push executor context toward limits. Executor quality degrades in the late tasks. Keep plans at 2–3 tasks.

## Flags
```bash
/gsd-execute-phase 1           # execute all plans in phase 1
/gsd-execute-phase 1 --plan 2  # re-run only plan 2 (after failure)
/gsd-execute-phase 1 --dry-run # show wave grouping without executing
```

## Example Workflow
```
/gsd-plan-phase 3       # produces plan-1.md, plan-2.md, plan-3.md
                         # plan-3 depends on plan-1

/gsd-execute-phase 3
# → Wave 1: plan-1 and plan-2 execute in parallel
# → Wave 2: plan-3 executes after Wave 1 completes
# → git log shows ~9 atomic commits

git log --oneline -10   # review commit trail
/gsd-verify-work 3      # verify what was built
```

## See Also
- [gsd-plan-phase](gsd-plan-phase.md) — produces the PLAN.md files consumed here
- [gsd-verify-work](gsd-verify-work.md) — verifies execution output
- [architecture/orchestration-model.md](../architecture/orchestration-model.md) — wave execution internals
