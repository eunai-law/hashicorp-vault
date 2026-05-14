# GSD Token Cost Analysis

## The Fundamental Economics

GSD is not token-efficient. It is quality-consistent. These are different goals that sometimes produce similar outcomes (long projects where context rot would have caused expensive rework) and sometimes don't (small tasks where the overhead exceeds the benefit).

**The question is not "does GSD save tokens?" — it doesn't.**  
**The question is "does GSD's quality improvement justify its token premium?"**

For some projects: yes. For others: no.

---

## Per-Command Cost Breakdown

### /gsd-new-project
| Component | Tokens | Notes |
|---|---|---|
| 4 parallel researchers | 40K–200K | Domain depth determines range |
| Research synthesizer | 15K–30K | Combines 4 outputs |
| Roadmapper | 10K–20K | Produces ROADMAP.md |
| **Total** | **65K–250K** | One-time; amortized across project |

### /gsd-discuss-phase
| Component | Tokens | Notes |
|---|---|---|
| Conversation | 2K–8K | No agents spawned |
| **Total** | **2K–8K** | Cheapest command |

### /gsd-plan-phase (per phase)
| Component | Tokens | Notes |
|---|---|---|
| SDK init | 1K–3K | |
| Phase researcher | 20K–50K | Web search adds cost |
| Pattern mapper | 15K–25K | Codebase read |
| Planner | 15K–30K | |
| Plan checker (×1) | 10K–20K | |
| Revision loops (×0–3) | 0–90K | 30K each |
| **Total** | **60K–220K** | High variance from revisions |

### /gsd-execute-phase (per phase, 4 plans × 3 tasks each)
| Component | Tokens | Notes |
|---|---|---|
| SDK init + wave grouping | 2K | |
| Wave 1: 3 executors × 40K | 120K | Parallel — all billed |
| Wave 2: 1 executor × 40K | 40K | |
| SUMMARY.md writes | 5K | |
| **Total** | **167K** | Scales linearly with plans × tasks |

### /gsd-verify-work
| Component | Tokens | Notes |
|---|---|---|
| SDK init | 1K | |
| Verifier agent | 15K–40K | Reads plans + code |
| **Total** | **16K–41K** | |

### /gsd-ship
| Component | Tokens | Notes |
|---|---|---|
| Code reviewer | 20K–60K | Scales with changed files |
| Code fixer (optional) | 0–40K | |
| PR creation | 1K | |
| **Total** | **21K–101K** | |

---

## Scenario Costs

### Scenario A: Small task (bug fix)

| Approach | Tokens | Quality |
|---|---|---|
| Direct prompt | 3K–8K | Good (fresh context) |
| `/gsd-quick` | 8K–20K | Good (fresh executor) |
| Full GSD loop | 250K–400K | Good (overkill) |

**Verdict:** Direct prompt or `gsd-quick`. The full loop costs 30–50× more for equivalent quality.

### Scenario B: Medium phase (new feature, ~2 weeks)

| Approach | Tokens | Quality |
|---|---|---|
| Single-thread, disciplined | 40K–100K | High early, degrades ~30% |
| GSD (balanced profile) | 300K–500K | Consistently high |
| GSD (budget profile) | 150K–250K | High with some quality tradeoff |

**Verdict:** GSD costs 3–5× more but delivers consistent quality. Worth it for features that matter.

### Scenario C: Large project (10 phases × 4 plans each)

| Approach | Tokens | Quality |
|---|---|---|
| Single-thread, accumulating | 200K–500K | Degrades severely in phases 6–10 |
| GSD (balanced) | 2M–4M | Consistent throughout |
| GSD (budget) | 800K–1.5M | Consistent, lower reasoning quality |

**Verdict:** GSD wins significantly for large projects. The context rot in a 10-phase single-thread project produces increasingly broken implementations that require expensive debugging.

---

## Hidden Cost Multipliers

### 1. Verification Failure → Re-execution (+50–150% of execute cost)
When verifier finds gaps, fix plans re-execute. A phase with significant verification failures can cost 2–2.5× the original execution. However: catching failure at verify time is cheaper than catching it 3 phases later when you've built on top of the broken foundation.

### 2. Plan Revision Loops (+60–90K per phase with 3 revisions)
Unpredictable but significant. An under-specified CONTEXT.md that leads to 3 revision cycles adds ~90K tokens to plan-phase cost. Write better CONTEXT.md to reduce this.

### 3. MCP Tool Schema Overhead
This is the biggest and most overlooked cost. Every active MCP server loads its tool schemas into the session context. 10 MCP servers × 200 tokens per tool schema = 2,000 tokens in every agent spawn. Across 40 agent spawns, that's 80,000 tokens purely from MCP overhead.

**Trim unused MCP servers before running GSD.**

### 4. Parallel Agent Context Initialization
8 parallel executors each initialize independently. Each reads PROJECT.md (~500 tokens), PLAN.md (~800 tokens), STATE.md (~300 tokens) = ~1,600 tokens per executor just for initialization. 8 × 1,600 = ~12,800 tokens before any work starts in a wave.

---

## Does GSD Reduce or Increase Token Spend?

**For a project below ~4 phases:** GSD increases token spend by 2–5× with moderate quality improvement. Probably not worth it unless execution quality is critical.

**For a project of 5–10 phases:** GSD increases token spend by 3–8× but prevents the quality collapse that would occur in a single-thread workflow. The rework cost avoided likely offsets the premium.

**For a project of 10+ phases:** GSD is cost-competitive when you account for single-thread rework costs in phases 7–10. A collapsed single-thread session that produces broken code costs tokens to debug and fix — those aren't counted in naive comparisons.

---

## Cost Reduction Strategies

Priority order (most impactful first):

1. **Trim MCP tool schemas** — disable unused MCP servers (biggest single lever per the GSD docs)
2. **Use `--profile=core`** — install only 6 core skills; removes baseline skill manifest overhead
3. **Use `budget` model profile** — Haiku for research + planning, Sonnet for verification
4. **Skip `plan_check`** for trivial phases — saves ~30K–50K per phase
5. **Use `/gsd-quick`** for tasks under 2 hours — saves ~200K–400K vs full loop
6. **Enable `dynamic routing`** — starts at Haiku, escalates only on failure
7. **Disable `research` toggle** when approach is clear from CONTEXT.md — saves ~35K–75K per phase
8. **Limit parallel executors** — `parallelization.max_concurrent: 3` reduces simultaneous context initialization cost
