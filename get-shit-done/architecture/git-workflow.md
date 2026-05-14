# GSD Git Workflow

## Why Git is Central

Git is not an afterthought in GSD — it is structural. Git provides:

1. **Atomic checkpoints**: Each task completion is a commit. Interrupted execution can always resume from the last commit.
2. **Execution audit trail**: The commit history documents exactly what was done, in what order, by which plan.
3. **Parallel conflict detection**: Wave-based execution with per-plan commits surfaces merge conflicts early, not at PR time.
4. **Reviewer-friendly history**: Clean, scoped commits make code review tractable on large phase outputs.

## Atomic Commit Protocol

Each executor creates **one git commit per task**, not one commit per plan. This is intentional:

```
feat(phase-1): create JWT validation middleware

feat(phase-1): apply auth middleware to protected routes

feat(phase-1): write integration tests for auth middleware

feat(phase-1): add rate limiting to login endpoint
```

Benefits:
- `git bisect` can find exactly which task introduced a regression
- Partial plan completion is visible and resumable
- Code reviewers can review task by task, not monolithic plan dumps

## Wave Execution and --no-verify

Within a wave, parallel executors use `git commit --no-verify`. This bypasses pre-commit hooks within the wave to avoid build-lock contention. Common problem: Rust's `cargo lock` file, which cannot be written by two processes simultaneously. If 8 executors all try to run `cargo build` in pre-commit hooks at the same time, they deadlock.

The orchestrator runs hooks **once after each wave completes**, not per-commit. This means:
- Hooks run; they just run once per wave instead of once per commit
- The no-verify commits are validated by the post-wave hook run
- Sequential execution (when parallelization is disabled) does not use `--no-verify`

## Branch Strategy

Two strategies, configured in `.planning/config.json`:

### Phase Strategy (default for most teams)
- Each phase gets a dedicated branch: `gsd/phase-1-auth`, `gsd/phase-2-user-profile`
- `/gsd-ship N` creates a PR from the phase branch to main
- Phases are merged sequentially; no cross-phase merges needed

### Milestone Strategy
- All phases within a milestone share a single branch: `gsd/milestone-1-mvp`
- Phases are merged into the milestone branch; one PR from milestone to main at the end
- Better for tightly coupled phases where cross-phase integration matters

Branch naming templates are configurable.

## /gsd-ship and PR Creation

```bash
/gsd-ship 1
```

Execution:
1. Spawns `gsd-code-reviewer` to review all changed files
2. Review findings are presented to user
3. Optionally spawns `gsd-code-fixer` for auto-fixable issues
4. Creates PR using `gh pr create`
5. PR body auto-generated from SUMMARY.md + VERIFICATION.md content

The `gsd-pr-branch` skill creates a **clean PR branch** by filtering `.planning/` commits from the branch. Planning artifacts (CONTEXT.md updates, STATE.md writes) do not appear in the PR diff.

## /gsd-complete-milestone

After all phases in a milestone are shipped and merged:

```bash
/gsd-complete-milestone
```

1. Archives the milestone's `.planning/phases/` into `.planning/archive/milestone-N/`
2. Creates a git tag: `v{major}.{minor}.0`
3. Updates STATE.md to reflect milestone completion
4. Runs `gsd-tools.cjs milestone complete <version>`

The archive preserves the full planning artifact trail for historical reference without cluttering the active `.planning/` directory.

## Checkpoint Recovery

If execution is interrupted (crash, manual cancel, token limit):

```bash
# See what was committed
git log --oneline --since="2 hours ago"

# Resume from where it stopped
/gsd-execute-phase 1 --plan 2  # re-run only plan 2
```

Or use STATE.md to determine position:
```bash
gsd-sdk query state json | jq '.current_plan'
```

## STATE.md as Git Supplement

STATE.md captures decisions and context that git cannot:

- Why a specific implementation approach was chosen
- What alternatives were considered and rejected
- External dependencies that blocked progress
- Session boundaries (when work stopped, why, what to do next)

This makes the commit history interpretable. A commit message like `feat(phase-1): use Redis for session storage` is explained by STATE.md's decision log entry: "Chose Redis over in-memory sessions — requirement REQ-14 specifies horizontal scalability."

## Worktrees for Parallel Workstreams

Advanced feature: GSD workstreams can use git worktrees to isolate parallel development areas:

```bash
/gsd-workstreams create --name api-v2 --worktree
```

Each workstream gets its own worktree and its own `.planning/` subdirectory. This allows truly parallel phase execution across independent areas without branch conflicts.

## Integration with CI

GSD does not directly integrate with CI systems, but the commit format is CI-friendly:
- Semantic commit messages support conventional-changelog tooling
- Phase branching creates clean PR boundaries for CI gate checks
- The `gsd-tools.cjs verify commits <hash1> <hash2>` command validates commit integrity post-execution
