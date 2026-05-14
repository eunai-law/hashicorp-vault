# /gsd-verify-work [N]

## Purpose
Performs goal-backward verification of a completed phase. Asks "Does what was built actually satisfy the requirements this phase was scoped to?" Produces a VERIFICATION.md with confirmed items, gaps, and structured fix plans for any failures.

## When to Use
- After `/gsd-execute-phase` completes
- Before `/gsd-ship` — verification is the quality gate before creating a PR
- After re-running gap fix plans (to confirm the gaps are now closed)

## Context Consumed
- All `.planning/phases/N/plan-{M}.md` — what was planned
- All `.planning/phases/N/summary-{M}.md` — what was actually done
- `.planning/REQUIREMENTS.md` — requirements scoped to this phase
- The actual codebase (agent reads relevant files to check implementation)
- `.planning/STATE.md` — current phase status

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/phases/N/VERIFICATION.md` | Per-requirement status (confirmed/gap), evidence, fix plans |

## Agents Spawned
One `gsd-verifier` in a fresh 200K-token context.

## Context Refresh
Yes — verifier starts fresh. It does not inherit the executor's context or decisions.

## Token Cost Profile
**Medium**

Single agent reads planning artifacts + relevant codebase files. Typical cost: 15K–40K tokens depending on phase size and depth of codebase reading.

Verification is usually the cheapest non-trivial command in the workflow. The cost of skipping it (discovering failures three phases later) is typically far higher.

## How Verification Works

The verifier runs **goal-backward analysis**:

1. Read REQUIREMENTS.md — identify all requirements scoped to this phase
2. For each requirement, find evidence in: SUMMARY.md, code files, tests
3. Mark as CONFIRMED (with file:line evidence), PARTIAL, or MISSING
4. For PARTIAL/MISSING: diagnose the specific gap and write a fix plan
5. Write VERIFICATION.md with full findings

The verifier also runs the verification commands from each PLAN.md:
```bash
npm test
npm run type-check
npm run e2e -- --grep auth
```

A plan verification command failure is treated as a gap even if the code appears correct.

## Common Pitfalls
- **Skipping verification**: "The tests pass" is not verification. Tests test implementation; verification checks requirements. These are different things.
- **Verification after stale artifacts**: If SUMMARY.md files are outdated (from a partial re-execution), the verifier works from incomplete evidence. Ensure all summary files are current.
- **Treating PARTIAL as acceptable**: A PARTIAL requirement means something was implemented but not completely. Don't ship PARTIAL items unless you've consciously decided to defer them.

## When Verification Finds Failures

VERIFICATION.md contains structured fix plans for each gap. The fix plan format is designed for immediate executor consumption — no planning cycle needed for simple gaps.

```
# From VERIFICATION.md:

### Gap: REQ-11 — Rate limiting on login endpoint
Fix Plan:
- Task: Apply rateLimiter middleware to POST /auth/login in src/routes/auth.ts:67
- Verification: curl -X POST /auth/login 201 times in 60 seconds → 429 on request 201
```

Re-execute: `/gsd-execute-phase N` (detects existing fix plans and routes them to executors)

## Example Workflow
```
/gsd-execute-phase 2     # executes all plans
/gsd-verify-work 2       # verifies results

# If failures found:
cat .planning/phases/2-*/VERIFICATION.md  # read gap details
/gsd-execute-phase 2     # re-run with fix plans
/gsd-verify-work 2       # re-verify

# Once clean:
/gsd-ship 2
```

## See Also
- [gsd-execute-phase](gsd-execute-phase.md) — what this verifies
- [gsd-ship](gsd-ship.md) — next step after clean verification
- [architecture/verification-system.md](../architecture/verification-system.md) — full verification internals
