# GSD Practical Usage Patterns

## Patterns That Work Well

### 1. The Full Loop for Complex Phases
```
/gsd-discuss-phase N    # lock decisions
/gsd-plan-phase N       # research → plan → verify
/gsd-execute-phase N    # parallel wave execution
/gsd-verify-work N      # goal-backward check
/gsd-ship N             # review + PR
```

**Works well when:** Phase has 3+ weeks of work, involves multiple independent workstreams, has meaningful architectural decisions, needs a clean git history for review.

**Evidence of effectiveness:** The fresh-context isolation prevents quality collapse in long phases. The atomic commit trail makes PR review tractable. The verification stage catches gaps before they compound.

### 2. Quick Mode for Small Tasks
```
/gsd-quick "Fix the validation bug in src/api/users.ts — empty name field returns 200, should return 422"
```

**Works well when:** Clear problem statement, no architectural decisions needed, under 2 hours of work.

**Anti-pattern:** Using `/gsd-quick` for anything requiring research or architectural choices. The executor starts fresh with no research context — it will make reasonable but possibly wrong choices.

### 3. Explore → Sketch → Discuss for UI Work
```
/gsd-explore "Options for the dashboard layout — cards vs table vs hybrid"
/gsd-sketch "Dashboard: user stats, recent activity, quick actions. 3 layout options."
# Review HTML mockups in browser
/gsd-discuss-phase N    # lock the chosen layout as a decision
/gsd-ui-phase N         # formalize the UI-SPEC.md
/gsd-plan-phase N       # plan with locked UI decisions
```

**Works well when:** Frontend phases with non-obvious layout choices. Sketch outputs are visual and fast; they surface options you wouldn't think to write in CONTEXT.md.

### 4. Brownfield Onboarding
```
/gsd-map-codebase       # analyze existing codebase (one-time)
/gsd-new-project        # researchers read .planning/codebase/ for context
/gsd-discuss-phase 1    # discuss with full codebase awareness
```

**Works well when:** Joining an existing project or inheriting a codebase. The codebase mapper gives researchers actual knowledge of existing patterns before any planning.

**Anti-pattern:** Running `/gsd-new-project` without `map-codebase` on a brownfield project. Researchers will generate generic plans that conflict with existing code patterns.

### 5. Workstreams for Parallel Teams
```
/gsd-workstreams create --name auth-system
/gsd-workstreams create --name payment-flow

# Developer A (auth-system branch):
/gsd-workstreams switch auth-system
/gsd-execute-phase 1

# Developer B (payment-flow branch, simultaneously):
/gsd-workstreams switch payment-flow
/gsd-execute-phase 2
```

**Works well when:** Multiple developers working on independent features. Each workstream has isolated STATE.md and phases, preventing cross-contamination.

### 6. Debug Without Destroying Context
```
/gsd-debug "Intermittent 500 on POST /checkout. Stack: [paste]. Happens ~1 in 50 requests."
```

**Works well when:** The bug is non-obvious and you don't want to burn your main session context with investigation. The debug session manager runs in isolated cycles, preserving your main context.

### 7. Session Continuity Protocol
```
# End of session:
/gsd-pause-work     # creates handoff artifact in STATE.md

# Start of next session:
/gsd-resume-work    # reads STATE.md, reconstructs position
/gsd-progress       # confirms next step
```

**Works well when:** Multi-day projects. The resume command eliminates the "where was I?" overhead at session start.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Full Pipeline for Trivial Tasks
**What it looks like:** Running `/gsd-discuss-phase`, `/gsd-plan-phase`, `/gsd-execute-phase`, `/gsd-verify-work` for a task that takes 20 minutes.

**Why it fails:** The pipeline cost (~300K tokens for a small phase) dwarfs the implementation cost. You'll spend more time waiting for agents than the implementation would have taken.

**Fix:** Use `/gsd-quick` for tasks under 2 hours. Reserve the full loop for phases with genuine complexity.

### Anti-Pattern 2: Skipping Discuss for Complex Phases
**What it looks like:** Going straight to `/gsd-plan-phase` for phases with ambiguous implementation choices.

**Why it fails:** The planner will make "reasonable default" choices. When you review the plan, you'll disagree with half of them. You'll either accept bad decisions or trigger multiple revision loops.

**Fix:** Always run `/gsd-discuss-phase` for phases with meaningful architectural choices. The 5K tokens it costs saves 90K+ in revision overhead.

### Anti-Pattern 3: Ignoring Verification Failures
**What it looks like:** Running `/gsd-verify-work`, seeing PARTIAL items, and running `/gsd-ship` anyway.

**Why it fails:** PARTIAL requirements shipped to production become bugs. The next phase builds on a broken foundation. The compounding rework cost is significant.

**Fix:** Fix PARTIAL items before shipping. The structured fix plans in VERIFICATION.md make this fast.

### Anti-Pattern 4: Not Trimming MCP Servers
**What it looks like:** Running GSD with 10+ MCP servers loaded, wondering why it's so expensive.

**Why it fails:** Each MCP server's tool schemas load into every agent context. 10 servers × 200 tokens × 40 agent spawns = 80,000 tokens of pure overhead.

**Fix:** Disable unused MCP servers before running GSD. This is the highest-leverage cost reduction available.

### Anti-Pattern 5: Large Plan Files
**What it looks like:** A PLAN.md with 8 tasks, all in one plan.

**Why it fails:** The executor accumulates context across all 8 tasks. By task 7, the executor is working in a context that's 40–50% full from prior task output. Quality degrades.

**Fix:** Keep plans to 2–3 tasks. Split large plans into multiple plans with dependency relationships.

### Anti-Pattern 6: Running Without Atomic State
**What it looks like:** Starting `/gsd-execute-phase` with uncommitted local changes.

**Why it fails:** Executor commits conflict with existing local changes. Merge conflicts appear, and the executor doesn't know how to resolve them.

**Fix:** Clean git state before execution. Commit or stash local changes first.

---

## When GSD Shines

- Projects spanning 4+ phases with genuine implementation complexity
- Teams where multiple developers share a `.planning/` directory
- Phases with parallel workstreams that can execute simultaneously
- Projects where the PR review trail matters (clean atomic commits)
- Work where context rot has been a visible problem in the past

## When GSD Adds Overhead Without Proportionate Benefit

- Single-feature additions to stable codebases
- Exploratory prototyping where you're not sure what you're building
- Fixes where the root cause is already known
- Solo developers with high AI prompting discipline who manage context manually
- Projects under 3 phases total
