# gsd-plan-checker

## Role
Validates PLAN.md files against 8 quality dimensions before approving them for execution. Can force planner revisions (up to 3 cycles). The quality gate between planning and execution.

## Inputs
- All `plan-{M}.md` files for the phase
- `CONTEXT.md` — locked decisions to verify compliance
- `REQUIREMENTS.md` — to verify coverage

## Outputs
- `PASS` verdict (orchestrator proceeds to execution)
- Structured revision request (specific dimensions to fix, for each failing plan)

## Lifecycle
1. Receives all PLAN.md files from orchestrator
2. Evaluates each plan against 8 dimensions
3. Emits PASS or detailed revision request
4. Orchestrator either proceeds or spawns a new planner in revision mode
5. Checker re-evaluates revised plans

## Orchestration Trigger
Spawned by `plan-phase` orchestrator after planner completes. May be spawned multiple times (up to 3 revision cycles).

## Isolation
**Fresh 200K-token context** per spawn.

**Tool access:** Read, Bash, Grep, Glob

## Token Behavior
**Medium** — reads plans and supporting artifacts. Typical: 10K–20K tokens per check cycle.

## The 8 Validation Dimensions

1. **Goal clarity** — Is the plan's goal stated precisely enough that completion is unambiguous?
2. **Task atomicity** — Are tasks small enough for a single executor pass? (Target: 2–3 tasks per plan)
3. **Dependency correctness** — Do stated wave dependencies match actual file-level dependencies?
4. **Coverage** — Does the plan address all requirements scoped to this phase?
5. **Verification commands** — Does every plan include runnable verification commands? (Nyquist gate)
6. **Out-of-scope discipline** — Is scope creep explicitly excluded?
7. **Pattern alignment** — Do implementation choices match existing codebase patterns (from PATTERNS.md)?
8. **CONTEXT.md compliance** — Does the plan honor all locked decisions?

## Escalation Behavior
After 3 failed revision cycles, the checker surfaces the blocking dimensions to the user with a diagnosis. Common resolutions: add more locked decisions to CONTEXT.md, split the phase scope, or manually adjust plans.
