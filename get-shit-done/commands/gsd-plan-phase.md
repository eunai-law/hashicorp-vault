# /gsd-plan-phase [N]

## Purpose
Converts a phase description + locked decisions into a set of atomic, executable PLAN.md files. Runs research, pattern analysis, planning, and plan-checking in sequence. The output drives `/gsd-execute-phase`.

## When to Use
- After `/gsd-discuss-phase` (or directly if the phase has no meaningful architectural choices)
- When you have a phase in ROADMAP.md and are ready to start implementation
- In `--gaps` mode after `/gsd-verify-work` finds failures (gap closure)
- In `--reviews` mode when cross-AI review feedback is available

## Context Consumed
- `.planning/phases/N/CONTEXT.md` — locked implementation decisions
- `.planning/REQUIREMENTS.md` — requirements scoped to this phase
- `.planning/PROJECT.md` — project vision and constraints
- `.planning/STATE.md` — current project state
- Codebase files (read by researcher and pattern mapper)

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/phases/N/RESEARCH.md` | Implementation approach, risks, key decisions from researcher |
| `.planning/phases/N/PATTERNS.md` | New file → existing analog mappings |
| `.planning/phases/N/plan-{M}.md` | One plan per parallel workstream (2–3 tasks each) |

## Agents Spawned

1. **gsd-phase-researcher** (fresh 200K context) — implementation research, risk analysis
2. **gsd-pattern-mapper** (fresh 200K context, parallel with researcher) — codebase pattern analysis
3. **gsd-planner** (fresh 200K context) — produces PLAN.md files from research output
4. **gsd-plan-checker** (fresh 200K context) — validates plans against 8 quality dimensions
5. **gsd-planner** (revision mode, 0–3 iterations) — if checker requests changes

## Context Refresh
Yes — all 4–5 agents run in fresh contexts. This is where context isolation pays the most dividends; planning quality does not degrade from prior session history.

## Token Cost Profile
**High**

Typical breakdown:
- Researcher: ~20K–40K tokens (reads codebase + external docs)
- Pattern mapper: ~15K–25K tokens (reads codebase)
- Planner: ~15K–25K tokens
- Plan checker: ~10K–15K tokens
- Per revision loop: ~25K–40K additional tokens

Total: ~60K–150K tokens per phase, depending on phase complexity and number of checker revisions.

**Cost reduction options:**
- `"workflow": { "research": false }` — skip researcher (use only if approach is clear)
- `"workflow": { "plan_check": false }` — skip checker (only for trivial phases)
- `--profile=budget` — use Haiku for research and planning

## Common Pitfalls
- **Undefined CONTEXT.md**: The planner makes assumption-driven decisions. Run `/gsd-discuss-phase` first for phases with meaningful choices.
- **Phase scope too large**: Planner produces plans with too many tasks per plan. Large plans exceed executor context or produce poor atomic commits. Keep phases to 2–4 weeks of work.
- **Checker loop exhaustion**: If checker cannot approve after 3 revisions, the phase scope is likely too ambiguous. Add more locked decisions to CONTEXT.md.
- **Stale research**: For phases planned weeks after research, patterns may have changed. Re-run `--research-only` to refresh.

## Flags
```bash
/gsd-plan-phase 1              # standard
/gsd-plan-phase 1 --gaps       # gap closure from VERIFICATION.md
/gsd-plan-phase 1 --reviews    # incorporate cross-AI review feedback
/gsd-plan-phase 1 --research-only  # refresh research without replanning
```

## Example Workflow
```
/gsd-discuss-phase 2    # lock decisions
/gsd-plan-phase 2       # research + plan + verify (auto-loops until approved)

# Review generated plans
ls .planning/phases/2-*/
cat .planning/phases/2-user-auth/plan-1.md
```

## See Also
- [gsd-discuss-phase](gsd-discuss-phase.md) — prerequisite for complex phases
- [gsd-execute-phase](gsd-execute-phase.md) — consumes PLAN.md files
- [architecture/planning-system.md](../architecture/planning-system.md) — full planning internals
