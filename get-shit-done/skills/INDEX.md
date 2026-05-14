# GSD Skills Index

Skills are composable capabilities installed into the AI runtime. Unlike full workflow commands, skills can be selectively installed via profiles and toggled at runtime with `/gsd:surface`.

## Core Workflow Skills (--profile=core)

| Skill | Capability | Type |
|---|---|---|
| [gsd-discuss-phase](gsd-discuss-phase.md) | Phase decision capture | Orchestration |
| [gsd-plan-phase](gsd-plan-phase.md) | Planning pipeline | Orchestration + agents |
| [gsd-execute-phase](gsd-execute-phase.md) | Wave execution | Orchestration + agents |
| [gsd-verify-work](gsd-verify-work.md) | Goal-backward verification | Orchestration + agents |
| [gsd-ship](gsd-ship.md) | Review + PR creation | Orchestration + agents |
| [gsd-progress](gsd-progress.md) | Position display + next-step | Standalone |

## Phase Management Skills (--profile=standard adds these)

| Skill | Capability | Type |
|---|---|---|
| [gsd-new-project](gsd-new-project.md) | Project initialization | Orchestration + agents |
| [gsd-new-milestone](gsd-new-milestone.md) | New milestone setup | Orchestration |
| [gsd-complete-milestone](gsd-complete-milestone.md) | Milestone archival + tagging | Orchestration |
| [gsd-phase](gsd-phase.md) | Phase CRUD | Standalone |
| [gsd-spec-phase](gsd-spec-phase.md) | Phase scope clarification | Standalone |
| [gsd-resume-work](gsd-resume-work.md) | Session context restoration | Standalone |
| [gsd-pause-work](gsd-pause-work.md) | Work handoff creation | Standalone |

## Exploration & Ideation Skills

| Skill | Capability | Type |
|---|---|---|
| [gsd-explore](gsd-explore.md) | Socratic ideation | Standalone |
| [gsd-sketch](gsd-sketch.md) | HTML mockup generation | Standalone |
| [gsd-spike](gsd-spike.md) | Feasibility experiments | Orchestration |
| [gsd-capture](gsd-capture.md) | Todo/backlog capture | Standalone |

## Quality Skills

| Skill | Capability | Type |
|---|---|---|
| [gsd-code-review](gsd-code-review.md) | Code review pipeline | Orchestration + agents |
| [gsd-secure-phase](gsd-secure-phase.md) | Security audit | Orchestration + agents |
| [gsd-ui-review](gsd-ui-review.md) | Visual audit | Orchestration + agents |
| [gsd-add-tests](gsd-add-tests.md) | Test generation | Orchestration + agents |
| [gsd-audit-fix](gsd-audit-fix.md) | Autonomous audit pipeline | Orchestration + agents |
| [gsd-debug](gsd-debug.md) | Bug investigation | Orchestration + agents |

## Navigation & Tooling Skills

| Skill | Capability | Type |
|---|---|---|
| [gsd-health](gsd-health.md) | Planning directory diagnostics | Standalone |
| [gsd-stats](gsd-stats.md) | Project statistics | Standalone |
| [gsd-update](gsd-update.md) | GSD self-update | Standalone |
| [gsd-undo](gsd-undo.md) | Safe undo | Standalone |
| [gsd-forensics](gsd-forensics.md) | Workflow post-mortem | Standalone |
| [gsd-settings](gsd-settings.md) | Interactive settings | Standalone |
| [gsd-config](gsd-config.md) | Direct config manipulation | Standalone |

## Namespace Router Skills (token-efficient entry points)

| Skill | Routes to | Token cost |
|---|---|---|
| [gsd-ns-workflow](gsd-ns-workflow.md) | discuss, plan, execute, verify, phase, progress | ~20 tokens |
| [gsd-ns-project](gsd-ns-project.md) | milestones, audits, summary | ~20 tokens |
| [gsd-ns-review](gsd-ns-review.md) | code review, debug, security, eval, UI | ~20 tokens |
| [gsd-ns-context](gsd-ns-context.md) | map, graphify, docs, learnings | ~20 tokens |
| [gsd-ns-manage](gsd-ns-manage.md) | config, workspace, workstreams, thread, ship | ~20 tokens |
| [gsd-ns-ideate](gsd-ns-ideate.md) | explore, sketch, spike, spec, capture | ~20 tokens |

## Advanced & Utility Skills

| Skill | Capability |
|---|---|
| [gsd-autonomous](gsd-autonomous.md) | Fully autonomous mode |
| [gsd-quick](gsd-quick.md) | Ad-hoc task execution |
| [gsd-fast](gsd-fast.md) | Fast mode toggle |
| [gsd-manager](gsd-manager.md) | Multi-phase dashboard |
| [gsd-workspace](gsd-workspace.md) | Workspace isolation |
| [gsd-workstreams](gsd-workstreams.md) | Parallel workstream management |
| [gsd-thread](gsd-thread.md) | Cross-session context threads |
| [gsd-map-codebase](gsd-map-codebase.md) | Codebase analysis |
| [gsd-graphify](gsd-graphify.md) | Knowledge graph |
| [gsd-docs-update](gsd-docs-update.md) | Documentation generation |
| [gsd-profile-user](gsd-profile-user.md) | Developer behavioral profile |
| [gsd-import](gsd-import.md) | External plan import |
| [gsd-ingest-docs](gsd-ingest-docs.md) | Document ingestion |
| [gsd-milestone-summary](gsd-milestone-summary.md) | Milestone summary |
| [gsd-pr-branch](gsd-pr-branch.md) | Clean PR branch creation |
| [gsd-review-backlog](gsd-review-backlog.md) | Backlog review |
| [gsd-inbox](gsd-inbox.md) | GitHub issue triage |

## Non-GSD Utility Skills (bundled)

| Skill | Capability |
|---|---|
| [revise-prompt](revise-prompt.md) | Restructure prompts for agent readability |
| [simplify](simplify.md) | Code quality review + simplification |
| [handoff](handoff.md) | Session handoff creation |
| [caveman](caveman.md) | Simple task description mode |
| [grill-me](grill-me.md) | Socratic questioning for idea validation |
| [write-a-skill](write-a-skill.md) | Create custom GSD skills |
| [init](init.md) | Project initialization utility |
| [loop](loop.md) | Recurring task execution |
| [schedule](schedule.md) | Scheduled remote agent management |
| [claude-api](claude-api.md) | Claude API / Anthropic SDK help |
| [update-config](update-config.md) | Claude Code settings configuration |
| [keybindings-help](keybindings-help.md) | Keyboard shortcut customization |
| [fewer-permission-prompts](fewer-permission-prompts.md) | Allowlist common tool calls |
| [security-review](security-review.md) | Security-focused code review |
