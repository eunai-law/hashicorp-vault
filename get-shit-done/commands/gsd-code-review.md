# /gsd-code-review

## Purpose
Reviews source files changed during a phase for bugs, security issues, and code quality problems. Produces a structured REVIEW.md with severity-classified findings. With `--fix`, auto-applies fixes for each finding atomically.

## When to Use
- Before creating a PR (also runs automatically as part of `/gsd-ship`)
- For standalone quality checks outside of the ship workflow
- With `--fix` to auto-remediate reviewable issues

## Context Consumed
- All files changed since phase start (or specified files)
- `.planning/phases/N/PLAN.md` files (for understanding implementation intent)

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/phases/N/REVIEW.md` | Severity-classified findings: CRITICAL / HIGH / MEDIUM / LOW |

## Agents Spawned
- **gsd-code-reviewer** — reads changed files, produces REVIEW.md
- **gsd-code-fixer** (with `--fix`) — applies fixes atomically, one commit per finding

## Context Refresh
Yes — both agents start fresh.

## Token Cost Profile
**Medium**

Scales with number of changed files. Typical: 20K–60K tokens for reviewer, additional 20K–40K for fixer.

## Severity Classification
- **CRITICAL**: Security vulnerabilities, data loss risks, auth bypasses
- **HIGH**: Logic bugs that affect correctness, missing error handling at boundaries
- **MEDIUM**: Code quality issues that could cause future bugs
- **LOW**: Style, naming, documentation gaps

## Flags
```bash
/gsd-code-review                # review all changed files
/gsd-code-review --fix          # review + auto-fix each finding
/gsd-code-review src/auth/      # review specific directory
/gsd-code-review --phase 2      # review phase 2 changes specifically
```

## Cross-AI Review
GSD supports cross-AI review via configured CLI integrations:
```bash
/gsd-review         # request review from configured external AI (Gemini, GPT, etc.)
```
External reviewer feedback can be incorporated back into planning with `--reviews` flag on `/gsd-plan-phase`.
