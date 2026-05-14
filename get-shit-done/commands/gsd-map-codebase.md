# /gsd-map-codebase

## Purpose
Analyzes an existing codebase in parallel across four focus areas (technology, architecture, quality, concerns) and produces structured intelligence documents in `.planning/codebase/`. Used before `/gsd-new-project` on brownfield projects, and to re-index after major refactors.

## When to Use
- Before `/gsd-new-project` on an existing codebase (gives researchers codebase context)
- After returning to a project after weeks/months (re-index for current state)
- When GSD's planning feels misaligned with actual codebase structure

## Context Consumed
- All codebase files (agents read selectively based on focus area)

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/codebase/tech.md` | Technology stack, dependencies, versions |
| `.planning/codebase/arch.md` | Architecture patterns, data flow, module structure |
| `.planning/codebase/quality.md` | Test coverage, code quality indicators, technical debt |
| `.planning/codebase/concerns.md` | Security concerns, performance hotspots, known issues |
| `.planning/intel/` | Queryable intelligence index for `--query` flag |

## Agents Spawned
Four parallel `gsd-codebase-mapper` agents, each with a different focus area.

## Context Refresh
Yes — all four agents run in fresh 200K contexts simultaneously.

## Token Cost Profile
**High (one-time)**

Four parallel agents reading across the codebase. Cost scales with codebase size:
- Small codebase (<10K LOC): ~60K–100K tokens
- Medium (10K–100K LOC): ~150K–300K tokens
- Large (100K+ LOC): ~300K–500K tokens

One-time cost (or occasional refresh). Amortized across a project, this is acceptable.

## Flags
```bash
/gsd-map-codebase              # full analysis
/gsd-map-codebase --query "authentication patterns"  # query existing intel
```

## Example Workflow
```
# Brownfield onboarding
/gsd-map-codebase       # analyze existing codebase
/gsd-new-project        # researchers read .planning/codebase/ for context
```
