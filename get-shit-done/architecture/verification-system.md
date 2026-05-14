# GSD Verification System

## What Verification Is

GSD's verification system is a **goal-backward analysis** of a completed phase. It asks: "Does what was built actually achieve what the phase promised, and does that achievement satisfy the requirements it was scoped to?"

Verification is architecturally separate from execution. It is not a code review, not a test runner, not a linter. It is a semantic check that the phase goal was met.

## Goal-Backward Analysis Explained

Standard analysis: "Here is what was implemented. Is it correct?"

Goal-backward analysis: "Here is what the phase required. Does the implementation prove these requirements are met?"

The difference matters. An implementation can pass all tests and fail goal-backward analysis if:
- Tests test the wrong things (testing implementation details, not behaviors)
- A requirement was misunderstood and implemented incorrectly
- A requirement was simply omitted (the executor skipped it)
- The implementation works but the integration point is broken

`gsd-verifier` starts from requirements, traces backwards to find evidence of their satisfaction in the implementation, and flags gaps.

## VERIFICATION.md Structure

```markdown
---
phase: 1
status: partial  # complete | partial | failed
verified_at: 2026-05-13T10:30:00Z
---

# Phase 1 Verification

## Summary
8/10 requirements verified. 2 gaps found.

## Verified Requirements
- [x] REQ-01: User registration endpoint — CONFIRMED (src/routes/auth.ts:45, test: auth.test.ts:12)
- [x] REQ-02: Password hashing — CONFIRMED (bcrypt, 12 rounds, auth.service.ts:23)
...

## Gaps Found

### Gap 1: REQ-08 — Email verification flow
**Status:** Missing
**Evidence:** No email service integration found. `sendVerificationEmail` referenced but not implemented.
**Fix Plan:** Implement SendGrid integration in src/services/email.ts. Wire into registration handler.
**Estimated complexity:** Medium (2–3 tasks)

### Gap 2: REQ-11 — Rate limiting on login endpoint
**Status:** Partial
**Evidence:** rate-limiter-flexible installed but middleware not applied to /auth/login route.
**Fix Plan:** Apply rateLimiter middleware to login route in src/routes/auth.ts:67.
**Estimated complexity:** Low (1 task)

## Verification Commands Run
```bash
npm test                    # 47 passed, 0 failed
npm run type-check          # 0 errors
npm run e2e -- --grep auth  # 12 passed, 1 failed (email verification)
```
```

## How It Differs from Tests

| Tests | Verification |
|---|---|
| Check implementation details | Check requirement satisfaction |
| Written by the implementer | Done by an independent agent |
| Automated | Semantic + automated |
| Run during development | Run after execution completes |
| Pass/fail per assertion | Pass/fail per requirement |
| Cannot catch wrong requirements | Catches wrong requirements |

Tests and verification are complementary. Tests prove the code does what it's supposed to do. Verification proves the code does what the *phase required*.

## When Verification Runs

The verification command (`/gsd-verify-work N`) is user-initiated, not automatic. GSD's design is:

1. Execute → artifacts produced
2. User decides when to verify
3. Verification can be manual (user tests features) or agent-driven (`gsd-verifier`)
4. Failures produce fix plans → re-execute

There is no automated CI hook that blocks shipping on verification failure. The discipline is workflow-driven, not enforcement-driven.

## Verification and UAT

`/gsd-audit-uat` cross-phase audit scans all `.planning/` phases for unresolved UAT items. It is separate from per-phase verification — it looks across the full project for outstanding acceptance criteria that have not been marked complete.

`gsd-tools.cjs audit-uat` powers this command, scanning all VERIFICATION.md files for `status: partial` or `status: failed`.

## Nyquist Validation

The Nyquist validation layer (planning-time) is a pre-execution complement to post-execution verification:

- At plan time: maps requirements to planned test coverage
- Blocks plans that have requirements with no corresponding verification commands
- Named for the Nyquist sampling theorem — sample rate must match signal frequency

This catches verification gaps before execution, not after. A plan without test coverage for a requirement cannot be approved.

## Relationship to Code Review

`/gsd-code-review` and `/gsd-verify-work` are different tools:

- **Verification** (`/gsd-verify-work`): Did we build the right thing? Requirements-driven.
- **Code review** (`/gsd-code-review`): Did we build it right? Quality-driven.

Both can find bugs but from different angles. A typical shipping sequence uses both:

```
/gsd-verify-work N   → requirements met?
/gsd-ship N          → spawns code reviewer → PR created
```

## Automated Fix Plans

When the verifier finds gaps, it writes structured fix plans directly into VERIFICATION.md. These fix plans are formatted to be immediately consumable by executors — they do not require the planner to re-run.

The fix plan format mirrors PLAN.md structure but is scoped to the specific gap. A gap like "middleware not applied to route" produces a 1-task fix plan that an executor can complete in minutes.

This is deliberate: the goal is zero manual debugging. The verifier diagnoses, the executor fixes.
