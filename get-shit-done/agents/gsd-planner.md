# gsd-planner

## Role
Creates executable PLAN.md files from research artifacts, CONTEXT.md decisions, and requirements. The planner's output is a set of plans that executors can implement without interpretation. Also runs in revision mode when the plan-checker requests changes.

## Inputs
- `RESEARCH.md` — phase research output (implementation approach, risks)
- `PATTERNS.md` — new file → existing analog mappings
- `CONTEXT.md` — locked implementation decisions (non-negotiable)
- `REQUIREMENTS.md` — requirements scoped to this phase
- `PROJECT.md` — project vision and constraints
- Plan-checker feedback (in revision mode)

## Outputs
- `plan-{M}.md` × N — one plan per parallel workstream (2–3 tasks per plan)

## Lifecycle
1. Receives structured payload from `plan-phase` orchestrator
2. Reads and prioritizes CONTEXT.md locked decisions
3. Identifies all must-have outcomes for the phase
4. Groups outcomes into parallel workstreams (minimal file-level dependencies between workstreams)
5. Assigns dependency waves across workstreams
6. Writes one PLAN.md per workstream
7. Returns control to orchestrator (which passes plans to plan-checker)
8. **Revision mode**: receives checker feedback, updates specific plans, returns for re-check

## Orchestration Trigger
Spawned by `plan-phase` orchestrator. Also spawned in `--gaps` mode and `--reviews` mode.

## Isolation
**Fresh 200K-token context** per spawn. In revision mode, the planner is a new instance that receives the original plans + checker feedback in its payload.

**Tool access:** Read, Write, Bash, Glob, Grep, WebFetch, mcp__context7__*

## Token Behavior
**Medium–High** — reads multiple artifact files, does substantial planning reasoning. Typical: 15K–30K tokens per planner instance. Revision cycles add 20K–30K each.

## Key Behaviors

### Goal-Backward Decomposition
The planner's core algorithm:
1. Start from phase goal (from ROADMAP.md)
2. Derive minimum must-have outcomes (what constitutes completion)
3. Group by parallel execution opportunity
4. Decompose into 2–3 atomic tasks per group
5. Assign dependency waves

### CONTEXT.md Compliance
Locked decisions from CONTEXT.md are **non-negotiable**. The planner reads them first and treats them as hard constraints. A plan that violates a locked decision will be rejected by the plan-checker.

### Plan Granularity Target
2–3 tasks per plan. This keeps each executor in a focused context, ensures atomic commits are meaningful, and prevents single-plan context exhaustion. Plans with 6+ tasks are split into multiple plans with a dependency relationship.

### "Plans are Prompts"
The planner is explicitly instructed: plans must be actionable as written, not require interpretation. Each task must specify:
- Exact files to create/modify
- Exact behavior to implement
- Pattern/analog to follow (from PATTERNS.md)
- Verification command to confirm correctness
