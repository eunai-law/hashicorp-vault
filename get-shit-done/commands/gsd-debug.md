# /gsd-debug

## Purpose
Scientific-method bug investigation using a multi-cycle debug session manager. Investigates bugs systematically: hypothesis → evidence → fix → verify. Manages checkpoint and continuation across multiple investigation cycles.

## When to Use
- When a bug's root cause is not immediately obvious
- When you've already tried the obvious fix and it didn't work
- When a bug involves multiple systems (e.g., auth + database + cache interaction)

## Context Consumed
- Bug description (provided inline)
- Codebase files (agent reads relevant areas)
- Error messages, stack traces, logs (if provided)
- `.planning/STATE.md` (for project context)

## Artifacts Created
- Debug session checkpoints (written to `.planning/debug/`)
- Structured fix plan (if root cause identified)
- Optional commit if fix is applied

## Agents Spawned
- **gsd-debug-session-manager** — manages the multi-cycle investigation loop
- **gsd-debugger** — executes individual investigation cycles (scientific method)
- Specialist agents as needed (e.g., gsd-code-reviewer for related code review)

## Context Refresh
Each debug cycle spawns a fresh `gsd-debugger`. The session manager maintains checkpoints in `.planning/debug/` to carry state between cycles without context inheritance.

## Token Cost Profile
**Medium–High (variable)**

Depends on bug complexity. Simple bugs: 1 cycle = ~20K tokens. Complex bugs: 3–5 cycles = ~100K–200K tokens. The session manager adds checkpoint overhead between cycles.

## Common Pitfalls
- **Providing insufficient context**: The debugger works best with a minimal reproduction case + error output. Vague descriptions ("it doesn't work") produce vague investigations.
- **Stopping after first hypothesis**: The session manager is designed for multi-cycle investigation. Don't exit after one cycle if the bug isn't resolved — continue.

## Example Workflow
```
/gsd-debug "POST /auth/login returns 500 intermittently. 
Error: 'Cannot read property userId of undefined' in auth.service.ts:145. 
Happens ~1 in 20 requests. Stack trace: [paste trace]"
```
