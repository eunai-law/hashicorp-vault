# gsd-quick

## Capability
Executes an ad-hoc task with GSD's execution guarantees (atomic commits, state tracking) while skipping the full discuss → plan → verify pipeline. For tasks small enough that planning overhead would exceed implementation time.

## Orchestration vs Execution
**Execution-capable** — spawns a single `gsd-executor` agent. No planning agents, no verification.

## How It Differs from Commands
`gsd-quick` is the lightweight alternative to the full `discuss → plan → execute → verify` loop. It foregoes research, planning artifacts, and verification in exchange for immediate execution of a clearly-scoped task.

Full pipeline: `~160K tokens for a medium phase`
gsd-quick: `~10K–30K tokens per task`

## Runtime Loading
Installed as a skill. Available in all sessions.

## Dependencies
Optional: reads `.planning/STATE.md` and `PROJECT.md` if present (for project context). Does not require CONTEXT.md or PLAN.md.

## Interactions
- Does not feed into verification pipeline
- Does create git commits (atomic per task)
- Does update STATE.md if present

## When to Use
- Bug fixes under 30 minutes of work
- Small feature additions clearly defined in one sentence
- Configuration changes, dependency updates, refactors with clear scope

## When NOT to Use
- Tasks requiring architectural decisions (use discuss + plan instead)
- Tasks with multiple independent workstreams (use execute-phase)
- Tasks where correctness is critical (use verify-work after)

## Example
```
/gsd-quick "Fix the 404 response — it currently returns HTML for API routes, should return JSON with {error: 'Not Found', code: 404}"

/gsd-quick "Update the Node.js version in package.json, Dockerfile, and .nvmrc to 22.4.0"

/gsd-quick "Add the missing CORS header 'Access-Control-Allow-Credentials: true' to the CORS middleware config"
```
