# GSD Strengths

Evidence-based analysis of what GSD does genuinely well, based on repository source and architecture.

## 1. Context Isolation Is Real and Mechanically Sound

**What it is:** Every executor, planner, researcher, and verifier starts with a clean 200K-token context window. This is not a "best practice recommendation" — it is enforced by architecture. Agents are spawned fresh; they cannot accumulate session history.

**Why it matters:** LLM quality degrades measurably above ~60–70% context utilization. Single-thread workflows that span multi-week projects inevitably cross this threshold. GSD's isolation prevents this from happening for any individual work unit.

**Evidence from repo:** The `gsd-executor.md` agent definition explicitly receives only a structured payload — PLAN.md + STATE.md + PROJECT.md — not conversation history. Context7 MCP calls use CLI fallback when MCP tools are stripped from agent contexts, showing attention to isolation edge cases.

## 2. Artifact-Driven State Survives Everything

**What it is:** All project state lives in `.planning/` as human-readable files. Session crashes, context resets, and runtime switches do not lose progress.

**Why it matters:** The single most common failure mode in AI-assisted development is losing work because you can't resume a session. GSD eliminates this entirely. STATE.md records exactly where work stopped; CONTEXT.md preserves decisions; SUMMARY.md captures what was done.

**Evidence from repo:** `gsd-tools.cjs state signal-waiting` writes structured resume signals. The `init` compound commands reconstruct full project context from artifacts in a single CLI call. `/gsd-resume-work` is a dedicated command for this scenario.

## 3. Spec-Before-Execution Prevents Wrong Implementations

**What it is:** CONTEXT.md locks decisions before planning. PLAN.md locks tasks before execution. Neither the planner nor the executor makes unconstrained architectural decisions.

**Why it matters:** The most expensive AI coding failure is implementing the right thing in the wrong way (e.g., using Redux when you wanted Zustand). This failure is invisible until code review — after significant execution cost. CONTEXT.md prevents it at the source.

**Evidence from repo:** The planner is explicitly instructed to parse CONTEXT.md first and treat locked decisions as non-negotiable. The plan-checker's dimension 8 specifically validates CONTEXT.md compliance. No other spec-driven system has this level of enforcement.

## 4. Wave-Based Parallelism with Dependency Safety

**What it is:** Plans are organized into dependency waves and executed in parallel within each wave. The orchestrator correctly waits for wave N before starting wave N+1.

**Why it matters:** Naive parallelism without dependency awareness creates merge conflicts and race conditions. GSD's wave model maximizes throughput while preventing shared-file conflicts.

**Evidence from repo:** STATE.md uses lockfile-based mutual exclusion for parallel writes. Git commits use `--no-verify` within waves and hook runs are deferred to post-wave. These are production-grade concurrency concerns, not theoretical.

## 5. Goal-Backward Verification Catches Requirement Gaps

**What it is:** The verifier starts from requirements, not from implementation. It asks "is there evidence that REQ-07 was satisfied?" rather than "does the code look correct?"

**Why it matters:** Code can pass all tests and fail requirements. The distinction between "code is correct" and "requirements are met" is lost in most AI workflows. GSD's verification stage restores this distinction.

**Evidence from repo:** VERIFICATION.md includes file:line evidence citations for CONFIRMED items. PARTIAL/MISSING items include structured fix plans. The Nyquist auditor at plan-time enforces that verification commands exist before execution even starts.

## 6. Token Cost Is Configurable Across a Wide Range

**What it is:** Model profiles (quality/balanced/budget/adaptive), toggle-able quality agents, skill profiles (core/standard/full), dynamic routing, and per-phase model overrides give fine-grained cost control.

**Why it matters:** A fixed-cost system would be unacceptable for small tasks. GSD allows users to pay precisely for the quality level they need per phase.

**Evidence from repo:** The configuration reference documents a 5-layer model resolution precedence. The `--profile=core` flag installs only 6 skills. Dynamic routing starts at Haiku and escalates only on soft failures.

## 7. Supply Chain Protection at Planning Time

**What it is:** The Package Legitimacy Gate (v1.51) runs `slopcheck` on AI-recommended packages. Packages with hallucinated names are flagged as [SLOP] before installation. Packages from web search are flagged as [ASSUMED] requiring human verification.

**Why it matters:** AI-hallucinated package names (slopsquatting) are a real attack vector. Malicious actors register packages with names that AI commonly hallucinates.

**Evidence from repo:** Three-layer protection: research layer flags [SLOP], planning layer adds human checkpoints before [ASSUMED] installs, execution layer blocks auto-fixes for package failures. This is unusually sophisticated supply chain awareness for a dev tool.

## 8. Runtime Agnosticism

**What it is:** GSD works across 14+ AI runtimes with runtime-specific installation, tool name translation, and model ID resolution.

**Why it matters:** Teams are not monolithic. Some developers use Claude Code, others use Cursor or Gemini CLI. A shared `.planning/` directory and runtime-agnostic artifact format means the team can use different runtimes without diverging project state.

**Evidence from repo:** Installer performs tool name mapping (`Bash` → runtime equivalent), hook event translation, and `resolve_model_ids: "omit"` for non-Claude runtimes. The artifact format (Markdown + JSON) is readable by any runtime.
