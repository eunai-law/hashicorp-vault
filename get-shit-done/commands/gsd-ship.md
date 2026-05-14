# /gsd-ship [N]

## Purpose
Creates a pull request from a verified phase. Spawns a code reviewer, optionally auto-fixes reviewable issues, then creates a PR with an auto-generated body derived from planning artifacts.

## When to Use
- After `/gsd-verify-work` returns a clean VERIFICATION.md
- When you're ready to merge phase work to main (or milestone branch)

## Context Consumed
- All files changed since phase start (for code review)
- `.planning/phases/N/VERIFICATION.md` — verification results (PR body source)
- `.planning/phases/N/summary-{M}.md` — execution summaries (PR body source)
- `.planning/config.json` — git branching strategy, PR template config

## Artifacts Created
| Artifact | Contents |
|---|---|
| Pull Request | Auto-generated body from VERIFICATION.md + SUMMARY.md |
| Clean PR branch | `.planning/` commits filtered out via gsd-pr-branch |
| REVIEW.md (optional) | Code review findings if `--save-review` flag used |

## Agents Spawned
1. **gsd-code-reviewer** (fresh 200K context) — reviews all changed files for bugs, security issues, quality problems
2. **gsd-code-fixer** (optional, fresh context) — auto-applies fixes for reviewer findings flagged as auto-fixable

## Context Refresh
Yes — both agents start fresh.

## Token Cost Profile
**Medium**

Code reviewer reads all changed files — cost scales with PR size. For a typical 5-plan phase:
- Reviewer: ~20K–50K tokens
- Fixer (if triggered): ~20K–40K tokens
- PR creation: minimal

Total: ~20K–90K tokens

## How gsd-pr-branch Works

The PR branch is created by filtering `.planning/` commits from the phase branch history. Planning artifact updates (CONTEXT.md writes, STATE.md updates, etc.) are excluded from the PR diff. Reviewers see only code changes.

```bash
# GSD creates a clean branch internally:
git checkout -b gsd/phase-1-auth-clean
# cherry-picks only non-.planning commits from gsd/phase-1-auth
```

## Common Pitfalls
- **Shipping without verify**: Running `/gsd-ship` before `/gsd-verify-work` skips the requirements check. The code reviewer catches quality issues but not requirements gaps.
- **Large PR with many unrelated changes**: If your phase scope was too broad, the PR becomes hard to review. Phase scoping during `/gsd-plan-phase` prevents this.
- **Code reviewer findings blocking ship**: The reviewer is advisory, not blocking. You can ship with known findings if you've triaged them. GSD does not enforce reviewer approval.

## PR Body Format

Auto-generated from artifacts:

```markdown
## Summary
[From VERIFICATION.md — what was built and verified]

## Requirements Addressed
- REQ-01: User authentication ✓
- REQ-02: Password hashing ✓
...

## Test Coverage
[From SUMMARY.md — test commands run and results]

## Known Gaps / Deferred Items
[From VERIFICATION.md partial/skipped items]

🤖 Generated with GSD + Claude
```

## Flags
```bash
/gsd-ship 1                   # standard: review + PR
/gsd-ship 1 --skip-review     # skip code review, just create PR
/gsd-ship 1 --save-review     # save review findings to REVIEW.md
/gsd-ship 1 --fix             # auto-fix reviewer findings before PR
```

## Example Workflow
```
/gsd-verify-work 3      # confirm clean verification
/gsd-ship 3             # code review + PR creation
# → PR URL printed
# → Review findings shown
```

## See Also
- [gsd-verify-work](gsd-verify-work.md) — prerequisite
- [gsd-code-review](gsd-code-review.md) — standalone code review without PR creation
- [gsd-pr-branch](gsd-pr-branch.md) — the skill that creates clean PR branches
