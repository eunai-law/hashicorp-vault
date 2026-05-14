# gsd-code-reviewer

## Role
Reviews source files changed during a phase for bugs, security vulnerabilities, and code quality problems. Produces REVIEW.md with severity-classified findings. Spawned automatically as part of `/gsd-ship`.

## Inputs
- All files changed since phase start (or specified files)
- `PLAN.md` files (for implementation intent context)

## Outputs
- `REVIEW.md` — findings classified as CRITICAL / HIGH / MEDIUM / LOW

## Lifecycle
1. Receives payload from code-review orchestrator or ship orchestrator
2. Reads all changed files
3. Reviews for: bugs, security issues, error handling gaps, type safety, injection vectors
4. Classifies each finding by severity
5. Writes REVIEW.md
6. Returns findings to orchestrator

## Orchestration Trigger
- Spawned by `code-review` command
- Spawned automatically by `ship` command before PR creation

## Isolation
**Fresh 200K-token context.**

**Tool access:** Read, Write, Bash, Grep, Glob

## Token Behavior
**Medium** — scales with number of changed files. Typical: 20K–60K tokens.

## Severity Classification
- **CRITICAL**: Auth bypass, SQL injection, data loss, secret exposure
- **HIGH**: Logic bugs, missing boundary validation, broken error paths
- **MEDIUM**: Code quality issues that may cause future bugs
- **LOW**: Style, naming, doc gaps

## Relationship to gsd-code-fixer
When `/gsd-code-review --fix` is used, `gsd-code-fixer` is spawned after the reviewer completes. The fixer reads REVIEW.md and applies fixes atomically (one commit per finding), targeting CRITICAL and HIGH severity items first.
