# GSD Token Management

## The Core Trade-Off

GSD does not reduce total token consumption. It trades **aggregate token cost** for **consistent quality**. Understanding this trade-off is essential to using GSD effectively.

In a single-thread workflow (no GSD), token consumption is front-loaded and cheap early, but quality degrades as the context fills. The last 30% of work happens in a degraded context.

In a GSD workflow, every agent starts fresh. Quality is consistent throughout. But every fresh start requires re-loading context — PROJECT.md, PLAN.md, STATE.md — which costs tokens on every spawn.

```
Single Thread:
Tokens: ████░░░░░░ (low early, quality good)
        ██████████ (high late, quality poor)

GSD:
Tokens: ████████████████████████ (consistently higher throughout)
Quality: ████████████████████████ (consistently good throughout)
```

## How GSD Manages Tokens

### 1. Namespace Routing (v1.40)

The biggest single token-saving mechanism. Instead of loading all 86 command descriptions into every session context (~2,150 tokens), GSD uses 6 namespace meta-skills (~120 tokens). This is a **94% reduction** in command-discovery overhead.

Direct command invocation (`/gsd-plan-phase`) bypasses routing entirely — zero overhead.

### 2. Structured Init Payloads

`gsd-tools.cjs init <workflow> <phase>` produces a compact JSON payload containing only the fields that workflow needs. An executor payload does not include research data. A planner payload does not include prior wave summaries.

When payloads exceed ~50KB, they're written to a temp file and returned as `@file:/tmp/gsd-init-XXXXX.json`, preventing large structured data from consuming the orchestrator's context.

### 3. Model Profiles

```json
{
  "model_profile": "balanced"
}
```

| Profile | Planning | Execution | Verification |
|---|---|---|---|
| `quality` | Opus | Sonnet | Opus |
| `balanced` | Sonnet | Sonnet | Sonnet |
| `budget` | Haiku | Haiku | Sonnet |
| `adaptive` | Starts Haiku, escalates on failure | | |
| `inherit` | Use runtime default | | |

**Dynamic routing** (`adaptive` mode): starts with cheaper models and escalates to premium only on soft failures. This can significantly reduce cost for phases where the work is straightforward.

### 4. Toggle-able Quality Agents

Research, plan checking, and verification can each be disabled:

```json
{
  "workflow": {
    "research": false,
    "plan_check": false,
    "verifier": false
  }
}
```

Disabling all three saves roughly 30–40% of plan-phase token cost. The trade-off is lower plan quality and no verification safety net.

### 5. MCP Tool Schema Trimming

The largest hidden token cost in Claude Code is unused MCP server tool schemas loaded into every session. If you have 10 MCP servers installed but only use 2, you're burning tokens on 8 sets of tool descriptions.

GSD's user guide identifies MCP schema trimming as the **biggest single lever** for reducing cost. Disable unused MCP tools in harness settings before tuning model profiles.

### 6. Skill Profiles

Install only the skills you need:

```bash
npx get-shit-done-cc@latest --profile=core       # 6 core-loop skills only
npx get-shit-done-cc@latest --profile=standard   # core + phase management
npx get-shit-done-cc@latest --profile=core,audit # composable
```

Fewer installed skills means fewer descriptions in the runtime's skill manifest, reducing baseline context overhead.

## Token Cost Profile by Operation

| Operation | Primary Cost Driver | Relative Cost | Notes |
|---|---|---|---|
| `/gsd-new-project` | 4 parallel researcher agents | **Very High** | One-time cost; amortized across project |
| `/gsd-discuss-phase` | Interactive — no agents | **Very Low** | Just conversation |
| `/gsd-plan-phase` | Researcher + Planner + Checker loop | **High** | Checker may force 2–3 planner revisions |
| `/gsd-execute-phase` | N executor agents × M tasks | **Very High** | Linear in number of plans × plan depth |
| `/gsd-verify-work` | 1 verifier agent | **Medium** | Single fresh context |
| `/gsd-ship` | 1 reviewer + PR creation | **Medium** | |
| `/gsd-quick` | Single agent, no planning | **Low** | For small tasks |
| `/gsd-code-review` | 1 reviewer + optional fixer | **Medium** | |
| `/gsd-map-codebase` | 4 parallel mapper agents | **High** | One-time brownfield analysis |
| `/gsd-debug` | Debug session + specialist agents | **Medium–High** | Depends on complexity |

## Hidden Token Costs

### 1. Plan Revision Loops

The plan-checker can force multiple planner revisions. Each revision is a full planner context + checker context cycle. A plan that requires 3 revisions pays 3× the planning token cost. Budget for 1–3 revisions on complex phases.

### 2. Agent Context Initialization

Every fresh agent must read its relevant artifacts before starting work. For an executor, this includes: PLAN.md (often 500–1000 tokens), PROJECT.md (~500 tokens), STATE.md (~300 tokens). This initialization overhead is paid on every agent spawn.

With 8 parallel executors in a wave, that's 8 × ~1,500 tokens = ~12,000 tokens just to initialize before any work begins.

### 3. Verification Failure → Re-execution

When `/gsd-verify-work` finds failures, it generates fix plans, which triggers another execution cycle. A phase with significant verification failures can cost 150–200% of the original execution cost.

### 4. Tool Schema Payload

Each agent has a `tools:` frontmatter that specifies allowed tools. Tools with complex schemas (like MCP tools) contribute their schema description to every agent prompt. Restricting tool access (as GSD does per-agent) partially mitigates this, but MCP tools still add overhead.

### 5. SUMMARY.md Production

Each executor writes a SUMMARY.md after completing its plan. The verifier reads all SUMMARY.md files. On large phases with many plans, this cross-plan reading adds up.

## Scenario Analysis

### Small Task (single bug fix)
- **With GSD** (`/gsd-quick`): ~5K–15K tokens. Single agent, no planning artifacts.
- **Without GSD** (direct prompt): ~2K–8K tokens.
- **Verdict**: GSD adds overhead. Use direct prompting or `/gsd-quick`.

### Medium Phase (new feature, 3–4 plans)
- **With GSD**: discuss (~1K) + plan (research 20K + plan 15K + check 10K) + execute (4 × 25K) + verify (15K) = ~160K tokens
- **Without GSD** (single thread): ~40K–80K tokens, but quality degrades in the last 30%
- **Verdict**: GSD costs 2–4× more but produces more consistent results. Worth it for features that matter.

### Large Project (10 phases × 4 plans each)
- **With GSD**: ~1.6M tokens across the project (40 execution cycles × ~40K avg)
- **Without GSD**: ~200K–400K tokens, but the last 5 phases produce poor output
- **Verdict**: GSD costs 4–8× more in aggregate. The quality difference is significant for complex projects.

## Comparison to Single-Thread Workflow

| Dimension | GSD | Single Thread |
|---|---|---|
| Early work quality | High | High |
| Late work quality | High (fresh context) | Low (context rot) |
| Total token cost | 3–8× higher | Baseline |
| Resumability | Full (artifacts persist) | Partial (session dependent) |
| Verification | Built in | Manual or none |
| Multi-developer | Yes (shared .planning/) | No |
| Cost predictability | Moderate | Low (depends on session length) |

## Does Verification Increase or Decrease Overall Spend?

**It increases spend in every individual case** (verification itself costs tokens). But it **decreases spend in aggregate** over a project:

- Catching a wrong implementation at verify time (15K tokens) costs far less than catching it 3 phases later when you've built on top of the error (fixing compound errors can cost 200K+ tokens).
- Verification is insurance against expensive rework. Skip it on low-risk plans, keep it on high-stakes implementations.

## Practical Cost Reduction Strategies

1. **Trim MCP tool schemas first** — biggest single lever
2. **Use `--profile=core`** if you only need the main loop
3. **Use `budget` model profile** for non-critical phases
4. **Disable `plan_check`** for trivial plans (CRUD, straightforward feature additions)
5. **Use `/gsd-quick`** for tasks under 2 hours of work
6. **Skip `research` toggle** when the approach is already clear from CONTEXT.md
7. **Direct command invocation** instead of namespace routing for known commands
