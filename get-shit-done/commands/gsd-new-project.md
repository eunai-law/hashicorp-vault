# /gsd-new-project

## Purpose
Transforms a project idea into a structured foundation: scoped requirements, a phased roadmap, and the persistent context artifacts that all subsequent GSD commands read. This is the project initialization command — you run it once at the start of a project.

## When to Use
- Starting a new project from scratch
- Starting a new major milestone on an existing project (`/gsd-new-milestone` for subsequent milestones)
- After running `/gsd-map-codebase` on a brownfield project (to give researchers full codebase context)

## Context Consumed
- User's answers to interactive questions (project vision, constraints, tech stack, target users)
- If `gsd-map-codebase` has been run: `.planning/intel/` and `.planning/codebase/` analysis files
- Existing `.planning/PROJECT.md` if present (for milestone continuations)

## Artifacts Created
| File | Contents |
|---|---|
| `.planning/PROJECT.md` | Vision, constraints, stakeholders, key decisions |
| `.planning/REQUIREMENTS.md` | Scoped, numbered requirements (REQ-01, REQ-02...) |
| `.planning/ROADMAP.md` | Phase breakdown with descriptions |
| `.planning/STATE.md` | Initialized project state |
| `.planning/config.json` | Default workflow configuration |

## Agents Spawned
Four parallel researcher agents, then a synthesizer, then a roadmapper:

1. **gsd-project-researcher** (×4, parallel): Domain research, market context, competitive analysis, technical requirements
2. **gsd-research-synthesizer**: Combines researcher outputs into a single SUMMARY.md
3. **gsd-roadmapper**: Converts synthesized research into ROADMAP.md with phased breakdown

## Context Refresh
Yes — all agents run in fresh 200K-token contexts. The main session stays thin.

## Token Cost Profile
**Very High (one-time)**

- 4 parallel researcher agents × ~50K tokens each = ~200K
- Synthesizer: ~30K tokens
- Roadmapper: ~20K tokens
- Total estimate: ~250K–350K tokens

Amortized across a full project, this is a small percentage of total cost. Worth paying once.

## Common Pitfalls
- **Running without map-codebase on brownfield projects**: Researchers won't know about your existing codebase. Run `/gsd-map-codebase` first.
- **Skipping the interactive questions**: The questions capture constraints that prevent bad planning decisions later. Don't rush through them.
- **Starting with too broad a scope**: GSD works better with a clear milestone boundary. "Build Twitter" produces a useless roadmap. "Build the authentication and user profile system" produces an actionable one.

## Example Workflow
```
# Brownfield project
/gsd-map-codebase       # analyze existing codebase
/gsd-new-project        # questions → research → roadmap

# Greenfield project
/gsd-new-project        # questions → research → roadmap

# Review output
cat .planning/ROADMAP.md
cat .planning/REQUIREMENTS.md
```

## See Also
- [gsd-new-milestone](gsd-new-milestone.md) — for starting a new version after completing a milestone
- [gsd-map-codebase](gsd-map-codebase.md) — brownfield codebase analysis
