# gsd-verifier

## Role
Performs goal-backward verification of a completed phase. Checks whether what was built satisfies the requirements the phase was scoped to. Produces VERIFICATION.md with per-requirement status and structured fix plans for any gaps.

## Inputs
- All `plan-{M}.md` files for the phase
- All `summary-{M}.md` files for the phase
- `REQUIREMENTS.md` — requirements scoped to this phase
- Relevant codebase files (reads to confirm implementation)
- `VERIFICATION.md` template (scaffolded by gsd-tools)

## Outputs
- `VERIFICATION.md` — per-requirement status (CONFIRMED/PARTIAL/MISSING), evidence with file:line citations, structured fix plans for gaps

## Lifecycle
1. Receives payload from `verify-work` orchestrator
2. Reads all planning artifacts
3. For each requirement: traces backwards from requirement → finds evidence in code/tests/summaries
4. Runs verification commands from PLAN.md files
5. Classifies each requirement as CONFIRMED (with evidence), PARTIAL (partially implemented), or MISSING
6. For PARTIAL/MISSING: writes structured fix plan
7. Writes VERIFICATION.md
8. Returns status to orchestrator

## Orchestration Trigger
Spawned by `verify-work` orchestrator after all executors complete.

## Isolation
**Fresh 200K-token context.** Does not inherit executor context or session history.

**Tool access:** Read, Write, Bash, Grep, Glob

## Token Behavior
**Medium** — reads multiple artifact files + relevant codebase sections. Typical: 15K–40K tokens. Scales with phase size.

## Key Behaviors

### Goal-Backward (not Forward) Analysis
The verifier does not ask "what was implemented, is it correct?" It asks "what was required, is there evidence of satisfaction?" This catches: wrong implementations, missing requirements, tests that test the wrong things.

### Evidence Requirement
Every CONFIRMED requirement needs a citation: `src/middleware/auth.ts:45` or `auth.test.ts:12`. Assumptions without evidence are marked PARTIAL.

### Fix Plan Format
Fix plans are structured for direct executor consumption — no planning cycle required for simple gaps:
```markdown
### Gap: [requirement title]
**Status:** MISSING
**Evidence:** [what was searched for and not found]
**Fix Plan:**
- Task: [specific implementation step]
- Files: [exact files to modify]
- Verification: [command to confirm fix]
**Complexity:** Low/Medium/High
```
