# GSD vs Other Frameworks

Technical comparison of GSD against BMAD, SpecKit, OpenSpec, and Taskmaster. Focused on operational mechanics, not marketing.

---

## GSD vs BMAD Method

**BMAD (Break it down, Make a plan, Analyze, Do)** is a prompting methodology for structured AI-assisted development. It emphasizes sequential thinking phases before implementation.

| Dimension | GSD | BMAD |
|---|---|---|
| **Orchestration** | Multi-agent, fresh contexts, wave execution | Single-thread prompting methodology |
| **Context management** | Architectural isolation (fresh agents) | User-managed (manual `/clear`, discipline) |
| **State persistence** | File-based (`.planning/`) | Conversation-based |
| **Verification** | Dedicated goal-backward analysis agent | Part of prompting discipline |
| **Planning** | Automated (planner agent + checker) | Manual (user writes the plan) |
| **Git integration** | Atomic commits per task, built in | Not specified |
| **Runtime support** | 14+ runtimes | Any LLM |
| **Token cost** | High (multi-agent overhead) | Lower (single thread) |
| **Learning curve** | High (40+ commands) | Low (it's a methodology) |

**Key difference:** BMAD is a mental model for how to prompt. GSD is an automated system that implements that mental model. BMAD is cheaper and more flexible for experienced developers. GSD is more reliable for teams or for complex multi-phase projects where BMAD discipline would otherwise slip.

---

## GSD vs SpecKit

**SpecKit** is a specification-driven development tool that generates specs from requirements and then implements from specs.

| Dimension | GSD | SpecKit |
|---|---|---|
| **Spec generation** | CONTEXT.md + PLAN.md (decisions + tasks) | Formal spec documents |
| **Orchestration** | Multi-agent parallel | Sequential single-agent |
| **Context isolation** | Yes — fresh agents | No — single thread |
| **Verification** | Built-in goal-backward | Manual |
| **Git workflow** | Atomic commits per task | Standard commits |
| **Artifact format** | Markdown + JSON | Structured spec format |
| **Runtime support** | 14+ | Primarily Claude |
| **Community** | Discord, 14 runtimes | Smaller |

**Key difference:** GSD puts more emphasis on execution orchestration (wave-based parallel, fresh contexts). SpecKit puts more emphasis on formal spec quality. GSD's specs (CONTEXT.md + PLAN.md) are simpler and more task-oriented; SpecKit's specs are more comprehensive and formal.

**User preference quote from GSD README:** *"I've done SpecKit, OpenSpec and Taskmaster — this has produced the best results for me."* — indicates GSD produces better execution results for at least some users.

---

## GSD vs OpenSpec

**OpenSpec** is a specification format and tooling for structured AI development, focused on creating detailed specifications before implementation.

| Dimension | GSD | OpenSpec |
|---|---|---|
| **Spec format** | Lightweight (CONTEXT.md + PLAN.md) | Comprehensive formal spec |
| **Implementation** | Automated multi-agent | Manual or single-agent |
| **Context management** | Architectural (fresh agents) | Spec-based (comprehensive context in spec) |
| **Verification** | Separate goal-backward agent | Part of spec review |
| **Flexibility** | Fixed pipeline | More flexible |
| **Overhead** | High for small tasks | Spec writing overhead |

**Key difference:** OpenSpec invests heavily in spec quality to give the AI full context upfront. GSD invests in process (research → plan → execute → verify) to produce quality iteratively. OpenSpec: better for well-understood domains. GSD: better for complex projects where research is needed.

---

## GSD vs Taskmaster

**Taskmaster** (AI Taskmaster) is an AI-powered task management system that decomposes projects into tasks and coordinates implementation.

| Dimension | GSD | Taskmaster |
|---|---|---|
| **Focus** | Full workflow (discuss → ship) | Task management + execution |
| **Context isolation** | Fresh agent per task | Varies |
| **Planning** | Research-driven, goal-backward | Task decomposition |
| **Verification** | Built-in requirements-based | Task completion tracking |
| **Git integration** | Deep (atomic commits, wave execution) | Standard |
| **State persistence** | `.planning/` directory | Task database |
| **Artifact richness** | CONTEXT.md, PLAN.md, SUMMARY.md, VERIFICATION.md | Task files |
| **Multi-phase support** | Yes (roadmap + milestone lifecycle) | Partial |

**Key difference:** Taskmaster is task-centric — it tracks tasks and coordinates their completion. GSD is workflow-centric — it manages the full software development lifecycle including decisions (discuss), research (plan), implementation (execute), quality (verify), and delivery (ship). GSD has more moving parts; Taskmaster is simpler.

---

## Summary Matrix

| Capability | GSD | BMAD | SpecKit | OpenSpec | Taskmaster |
|---|---|---|---|---|---|
| Fresh context isolation | ✅ | Manual | ❌ | ❌ | Partial |
| Automated planning | ✅ | ❌ | ✅ | ❌ | ✅ |
| Goal-backward verification | ✅ | ❌ | Partial | Partial | ❌ |
| Atomic git commits | ✅ | Manual | ❌ | ❌ | ❌ |
| Multi-runtime | ✅ (14+) | Any | Limited | Limited | Limited |
| Phase + milestone lifecycle | ✅ | ❌ | ❌ | ❌ | Partial |
| Supply chain protection | ✅ | ❌ | ❌ | ❌ | ❌ |
| Session continuity | ✅ | Manual | Partial | ❌ | ✅ |
| Token cost | Very High | Low | Medium | Medium | Medium |
| Learning curve | High | Low | Medium | Medium | Medium |

## When to Choose Each

| Project type | Recommendation |
|---|---|
| Simple features, experienced AI developer | BMAD or direct prompting |
| Complex multi-phase project, team use | GSD |
| Well-understood domain, need detailed specs | SpecKit or OpenSpec |
| Task tracking + light orchestration | Taskmaster |
| Cross-runtime team | GSD (only option with 14+ runtime support) |
