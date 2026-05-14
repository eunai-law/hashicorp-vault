# /gsd-discuss-phase [N]

## Purpose
Captures your implementation decisions for a specific phase before planning begins. Converts the one-line phase description in ROADMAP.md into a set of locked decisions that the planner must honor. This prevents the planner from making architectural choices you'd disagree with.

## When to Use
- Before every `/gsd-plan-phase` call, especially for phases with ambiguous implementation choices
- When you have strong opinions about: API shape, data model, UI layout, library choice, error handling approach, integration patterns
- Skip it if the phase is straightforward with no meaningful architectural choices (CRUD endpoints with clear patterns, simple configuration)

## Context Consumed
- `.planning/ROADMAP.md` — phase description to identify gray areas
- `.planning/STATE.md` — current project context and prior decisions
- `.planning/PROJECT.md` — project constraints and vision

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/phases/N-{name}/CONTEXT.md` | Locked decisions, each marked as non-negotiable for the planner |

CONTEXT.md example:
```markdown
# Phase 1 Context

## Locked Decisions

**Authentication:** Use JWT with RS256 algorithm. Private key loaded from PRIVATE_KEY env var.
_Reason: Existing infrastructure uses RS256; symmetric keys rejected in security review._

**Session storage:** Stateless — no server-side sessions. JWT carries all needed claims.

**Error format:** RFC 7807 Problem Details JSON for all 4xx/5xx responses.
```

## Agents Spawned
None. This is a pure conversation — you and the orchestrator, no subagents.

## Context Refresh
No agents spawned; no context isolation needed. The orchestrator runs in your current session.

## Token Cost Profile
**Very Low**

Interactive conversation only. Typically 2K–5K tokens for the full discussion. The cheapest command in the GSD workflow.

## Common Pitfalls
- **Too vague decisions**: "Use good patterns" is not a locked decision. "Follow the pattern in `src/middleware/logging.ts`" is.
- **Skipping discuss for complex phases**: The planner will make reasonable but wrong choices. You'll discover this after execution, not before.
- **Over-specifying trivial phases**: Spending 20 minutes discussing a 1-task phase that has no meaningful choices wastes time.

## Example Workflow
```
/gsd-discuss-phase 3

# GSD asks about:
# - "The spec mentions 'real-time updates' — WebSockets or SSE?"
# - "Error handling: fail silently or propagate to user?"
# - "State management: local component state or global store?"

# You answer each question.
# GSD writes CONTEXT.md with your answers as locked decisions.
# Then:

/gsd-plan-phase 3   # planner reads CONTEXT.md and honors your decisions
```

## See Also
- [gsd-spec-phase](gsd-spec-phase.md) — more formal spec process with ambiguity scoring
- [gsd-plan-phase](gsd-plan-phase.md) — the planning command that reads CONTEXT.md
