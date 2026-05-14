# /gsd-quick

## Purpose
Executes an ad-hoc task with GSD's execution guarantees (atomic commits, state tracking) but skips the full discuss → plan → verify pipeline. For tasks too small to justify full orchestration overhead.

## When to Use
- Small, well-understood tasks (bug fixes, minor feature additions, refactors under 2 hours)
- When you don't need planning artifacts — the task is clear enough to describe in one sentence
- When you want GSD's commit discipline without the planning overhead

## Context Consumed
- Task description (provided inline)
- Codebase context (agent reads relevant files)
- `.planning/STATE.md` (if present) — for project context

## Artifacts Created
- Code implementation
- One or more atomic git commits
- Optional: brief SUMMARY.md entry

## Agents Spawned
Single `gsd-executor` in a fresh 200K-token context.

## Context Refresh
Yes — single fresh agent context.

## Token Cost Profile
**Low**

Single executor, no planning research, no verification. Typically 10K–30K tokens.

This is the cheapest GSD command that does real work. Use it aggressively for small tasks instead of the full pipeline.

## Common Pitfalls
- **Using quick for large tasks**: "Build the auth system" is not a quick task. Quick is for "add email validation to the registration form."
- **No verification**: Quick skips `/gsd-verify-work`. If correctness matters, run verify manually or use the full pipeline.
- **Missing context**: The executor starts fresh and reads the codebase, but if your task requires understanding prior decisions from CONTEXT.md or STATE.md, those artifacts are still loaded.

## Example Workflow
```
/gsd-quick "Add input validation to the POST /users endpoint — reject empty email and name fields with 422"

# GSD spawns a single executor
# → reads relevant route files
# → implements validation
# → commits: "feat: add input validation to POST /users endpoint"
```
