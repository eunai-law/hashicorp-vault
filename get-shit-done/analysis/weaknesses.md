# GSD Weaknesses

Honest analysis of real limitations and failure modes. Based on architectural analysis of the repository.

## 1. Aggregate Token Cost Is High — Often 3–8× Single-Thread

**The problem:** Fresh-context isolation prevents quality decay but does not reduce total token consumption. It increases it. Every agent spawn re-loads context that would persist for free in a single-thread workflow. A 10-phase project with 4 plans per phase involves ~40 full executor contexts, each loading PROJECT.md, PLAN.md, and STATE.md independently.

**When it hurts most:** Small-to-medium projects where the full planning → execution → verification pipeline costs more tokens than the implementation would have cost in a focused single-thread session with a disciplined developer.

**Mitigation:** `gsd-quick` for small tasks, `budget` model profile, skill profiles, MCP trimming. But these are dials, not solutions — the fundamental cost structure is higher than alternatives.

**Honest comparison:** A developer who uses Claude Code directly with careful context management and `/clear` at appropriate moments can match GSD quality at lower cost for projects under 4–6 phases.

## 2. Planning Overhead Is Too Heavy for Small Tasks

**The problem:** The `discuss → plan → execute → verify` pipeline involves 4–6 agent spawns before a single line of code is written. For a bug fix or small feature addition, this overhead is disproportionate.

**When it hurts most:** Teams that try to use the full GSD loop for everything, including trivial tasks. This is a user behavior problem that GSD partially addresses with `gsd-quick`, but the full loop is the "recommended" experience and many users default to it.

**The trap:** GSD's documentation emphasizes the full loop. New users over-engineer small work. The overhead becomes the experience rather than the quality.

## 3. Cold-Start Artifact Dependency Creates Fragility

**The problem:** Every agent reads planning artifacts from `.planning/`. If those artifacts are stale, incorrect, or missing, agents make bad decisions with high confidence. A CONTEXT.md that doesn't match current codebase reality produces plans that are coherent but wrong.

**When it hurts most:** Long-running projects where the codebase evolves but planning artifacts aren't updated. A CONTEXT.md written in week 1 may be misleading in week 8 after significant refactoring.

**Mitigation:** `/gsd-map-codebase` for periodic re-indexing. But this is a manual step that many users skip.

## 4. Orchestration Debugging Is Opaque

**The problem:** When the system misbehaves — a plan-checker that loops infinitely, an executor that makes wrong choices, a verifier that misclassifies gaps — diagnosing the problem requires understanding the full agent hierarchy. The user sees a command that "didn't work." Understanding why requires reading PLAN.md, SUMMARY.md, VERIFICATION.md, STATE.md, and potentially agent-level outputs.

**When it hurts most:** Teams without deep GSD familiarity. `/gsd-forensics` exists for this, but it adds another layer of complexity.

**Comparison:** A single-thread workflow failure is directly observable in the conversation. A GSD failure may be buried in a SUMMARY.md from a completed agent that no longer exists.

## 5. Spec Lock-In Reduces AI Flexibility

**The problem:** CONTEXT.md's locked decisions prevent the AI from making better choices discovered during implementation. If your locked decision turns out to be wrong (you chose SSE over WebSockets, then discovered WebSockets were required for your use case), the executor will implement the wrong thing without questioning it.

**When it hurts most:** Exploratory phases where the right approach isn't known in advance. The spec-driven model assumes you know what you want before planning. For genuinely exploratory work, this assumption fails.

**Mitigation:** `/gsd-explore` and `/gsd-spike` before discuss-phase. But these add time and tokens.

## 6. Non-Claude Runtime Support Is Second-Class

**The problem:** GSD is built for and tested primarily on Claude Code. Non-Claude runtimes (Gemini, Codex, Cursor, Windsurf) have documented capability gaps: no worktrees, tool name mismatches, no hook system, different agent spawning semantics.

**When it hurts most:** Teams that adopt GSD across mixed runtimes. Features that work perfectly in Claude Code may silently degrade or fail in Gemini CLI.

**Evidence:** The installer sets `resolve_model_ids: "omit"` for non-Claude runtimes and the architecture notes tool name translation. These are workarounds, not parity.

## 7. Plan Revision Loops Can Multiply Cost

**The problem:** The plan-checker can force up to 3 planner revisions. Each revision is a full planner spawn (~20K–30K tokens) plus a full checker spawn (~10K–20K tokens). A phase that requires 3 revisions costs ~2.5× the baseline planning cost.

**When it hurts most:** Complex phases with under-specified CONTEXT.md, or phases where the checker's standards are high (all 8 dimensions strict). The revision count is unpredictable before planning runs.

**Mitigation:** Better CONTEXT.md specificity reduces revisions. Disabling `plan_check` toggle eliminates the overhead but removes the quality gate.

## 8. Surface Area Is Large and Growing

**The problem:** 40+ skills, 31 agents, 86 commands, 14 runtimes, multiple configuration layers, a CLI toolchain, an SDK, and an ongoing changelog. The system complexity is high and growing. The "simple 6-command loop" advertised in the README is accurate for the happy path but understates the cognitive overhead of understanding when to use which command and how to recover from failures.

**When it hurts most:** New users who encounter an error and don't know whether to run `/gsd-health`, `/gsd-forensics`, `/gsd-resume-work`, or `gsd-sdk query state validate`. The error recovery surface is complex.
