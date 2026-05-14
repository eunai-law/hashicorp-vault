# GSD Execution Lifecycle

## Full Sequence: discuss → plan → execute → verify → ship

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant SDK as gsd-tools / SDK
    participant A as Agents
    participant FS as .planning/
    participant GIT as Git

    Note over U,GIT: PHASE SETUP
    U->>O: /gsd-discuss-phase 1
    O->>SDK: init phase-op 1
    SDK-->>O: project context + phase info
    O->>U: Interactive questions (gray areas)
    U->>O: Answers
    O->>FS: Write CONTEXT.md (locked decisions)
    O->>SDK: state add-decision (for each answer)

    Note over U,GIT: PLANNING
    U->>O: /gsd-plan-phase 1
    O->>SDK: init plan-phase 1
    SDK-->>O: PROJECT + REQUIREMENTS + CONTEXT
    O->>A: Spawn gsd-phase-researcher (fresh ctx)
    O->>A: Spawn gsd-pattern-mapper (fresh ctx, parallel)
    A-->>FS: Write RESEARCH.md
    A-->>FS: Write PATTERNS.md
    O->>A: Spawn gsd-planner (reads RESEARCH + CONTEXT)
    A-->>FS: Write PLAN.md × N
    O->>A: Spawn gsd-plan-checker
    A-->>O: Checker verdict (PASS or revise)
    alt Checker requests revision
        O->>A: Spawn gsd-planner (revision mode)
        A-->>FS: Update PLAN.md files
        O->>A: Spawn gsd-plan-checker again
    end
    O->>SDK: state update (planning complete)

    Note over U,GIT: EXECUTION
    U->>O: /gsd-execute-phase 1
    O->>SDK: init execute-phase 1
    SDK-->>O: All PLAN.md files + wave grouping
    O->>A: Spawn Wave 1 executors (parallel)
    loop Per executor in wave
        A->>FS: Read PLAN.md
        A->>GIT: Implement task 1 → git commit
        A->>GIT: Implement task 2 → git commit
        A->>GIT: Implement task 3 → git commit
        A-->>FS: Write SUMMARY.md
        A-->>SDK: state update-progress
    end
    Note over O,A: Wait for all Wave 1 to complete
    O->>A: Spawn Wave 2 executors (parallel, if any)
    O->>SDK: state begin-phase (mark executed)

    Note over U,GIT: VERIFICATION
    U->>O: /gsd-verify-work 1
    O->>SDK: init verify-work 1
    SDK-->>O: PLAN.md + SUMMARY.md + REQUIREMENTS
    O->>A: Spawn gsd-verifier (fresh ctx)
    A->>FS: Goal-backward analysis
    A-->>FS: Write VERIFICATION.md
    alt Failures found
        A-->>O: Fix plans
        O->>U: Present failures + fix plans
        U->>O: Approve re-execution
        O->>A: Spawn executors for fix plans
    end

    Note over U,GIT: SHIPPING
    U->>O: /gsd-ship 1
    O->>A: Spawn gsd-code-reviewer
    A-->>O: Review findings
    O->>GIT: Create PR (auto-generated body)
    O->>U: PR URL
```

## Step-by-Step Detail

### Step 1: /gsd-discuss-phase N

**What it reads:** ROADMAP.md (phase description), STATE.md (current context), PROJECT.md

**What it writes:** `phases/N-{name}/CONTEXT.md`

**How it works:** The command is interactive — it asks you targeted questions about implementation decisions specific to this phase. Gray areas identified from the roadmap description become questions. Your answers are written as "locked decisions" with explicit markers that planners must honor.

No agents are spawned. This is a conversation, not an orchestration.

**Git impact:** None. Context is artifact-only.

---

### Step 2: /gsd-plan-phase N

**Agents spawned:**
1. `gsd-phase-researcher` — reads CONTEXT.md and codebase, researches implementation approach
2. `gsd-pattern-mapper` — reads codebase, maps new files to existing analogs (parallel with researcher)
3. `gsd-planner` — reads RESEARCH.md + PATTERNS.md + CONTEXT.md, produces PLAN.md files
4. `gsd-plan-checker` — validates plans against 8 dimensions
5. `gsd-planner` (revision) — if checker requests changes (0–3 iterations)

**What it reads:** PROJECT.md, REQUIREMENTS.md, CONTEXT.md, codebase files

**What it writes:**
- `phases/N/RESEARCH.md`
- `phases/N/PATTERNS.md`  
- `phases/N/plan-{M}.md` (one per parallel workstream)

**Git impact:** None. Planning is pre-execution.

**Token cost:** High. This is typically the most expensive step after execution itself.

---

### Step 3: /gsd-execute-phase N

**Agents spawned:** One `gsd-executor` per plan, grouped into dependency waves.

**What it reads:** All `plan-{M}.md` files, STATE.md, PROJECT.md

**What each executor writes:**
- Code files (implementation)
- `phases/N/summary-{M}.md`
- Updates STATE.md (via lockfile-protected writes)

**Git impact:** Major. Each executor creates one commit per task using the format:
```
feat(phase-1): implement user authentication middleware

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Wave commits use `--no-verify` to avoid build-lock contention; the orchestrator runs hooks once after each wave.

---

### Step 4: /gsd-verify-work N

**Agents spawned:** `gsd-verifier`

**What it reads:** All plan-{M}.md files, all summary-{M}.md files, REQUIREMENTS.md, the actual codebase

**What it writes:** `phases/N/VERIFICATION.md`

**How it works:** Goal-backward analysis. The verifier asks: "Does what was built actually achieve what the plan promised? Does the plan achievement satisfy the requirements it was scoped to?"

If failures are found: generates fix plans in VERIFICATION.md format, ready for re-execution without manual debugging.

---

### Step 5: /gsd-ship N

**Agents spawned:** `gsd-code-reviewer`, optionally `gsd-code-fixer`

**What it reads:** All files changed since phase start, VERIFICATION.md

**What it writes:** Pull request (via `gh pr create`)

**How it works:** The `gsd-pr-branch` skill creates a clean PR branch by filtering `.planning/` commits from the branch history. The PR body is auto-generated from VERIFICATION.md and SUMMARY.md artifacts.

## Error Recovery

### Interrupted Execution
If an executor fails mid-plan: git history shows exactly how far it got (atomic commits). Resume with `/gsd-execute-phase N --plan M` to re-run only the failed plan.

### Verification Failures
VERIFICATION.md contains structured fix plans. Run `/gsd-execute-phase N` again — the orchestrator detects existing fix plans and routes them to executors rather than re-running completed plans.

### State Drift
If `.planning/STATE.md` gets out of sync with the actual codebase:
```bash
gsd-sdk query state validate
gsd-sdk query state sync
```

Or use `/gsd-health --repair` for interactive repair.

### Plan Quality Failures
If the plan-checker cannot approve a plan after 3 revisions, it surfaces the blocking dimensions to the user. Common causes: under-specified CONTEXT.md (add more decisions), overly complex phase scope (split the phase).
