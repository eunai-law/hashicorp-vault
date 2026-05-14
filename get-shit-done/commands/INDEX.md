# GSD Commands Index

All GSD commands with purpose, token cost tier, and artifact summary.

## Core Workflow Loop

| Command | Purpose | Token Cost | Key Artifact |
|---|---|---|---|
| [gsd-new-project](gsd-new-project.md) | Initialize project: research → requirements → roadmap | Very High (one-time) | PROJECT.md, REQUIREMENTS.md, ROADMAP.md |
| [gsd-discuss-phase](gsd-discuss-phase.md) | Capture implementation decisions before planning | Very Low | CONTEXT.md |
| [gsd-plan-phase](gsd-plan-phase.md) | Research + plan + verify loop | High | PLAN.md × N |
| [gsd-execute-phase](gsd-execute-phase.md) | Execute plans in parallel waves | Very High | Code + SUMMARY.md × N |
| [gsd-verify-work](gsd-verify-work.md) | Goal-backward phase verification | Medium | VERIFICATION.md |
| [gsd-ship](gsd-ship.md) | Code review + PR creation | Medium | Pull Request |
| [gsd-progress](gsd-progress.md) | Show current position + next step | Low | None |
| [gsd-complete-milestone](gsd-complete-milestone.md) | Archive milestone + tag release | Low | Git tag + archive |
| [gsd-new-milestone](gsd-new-milestone.md) | Start next milestone cycle | Medium | Updated ROADMAP.md |

## Phase Management

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-phase](gsd-phase.md) | CRUD for phases in ROADMAP.md | Low |
| [gsd-spec-phase](gsd-spec-phase.md) | Clarify phase scope before discussing | Low–Medium |
| [gsd-ui-phase](gsd-ui-phase.md) | Generate UI-SPEC.md design contract | Medium |
| [gsd-mvp-phase](gsd-mvp-phase.md) | Plan phase as vertical MVP slice | Medium |
| [gsd-ai-integration-phase](gsd-ai-integration-phase.md) | Design AI system spec (AI-SPEC.md) | High |
| [gsd-validate-phase](gsd-validate-phase.md) | Validate phase goal achievement | Medium |
| [gsd-ultraplan-phase](gsd-ultraplan-phase.md) | Cloud-offloaded plan generation (beta) | High |

## Quality & Review

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-code-review](gsd-code-review.md) | Review changed files for bugs/security/quality | Medium |
| [gsd-secure-phase](gsd-secure-phase.md) | Verify threat model mitigations | Medium |
| [gsd-ui-review](gsd-ui-review.md) | Retroactive 6-pillar visual audit | Medium |
| [gsd-eval-review](gsd-eval-review.md) | Audit AI evaluation coverage | Medium |
| [gsd-add-tests](gsd-add-tests.md) | Generate tests based on UAT criteria | Medium |
| [gsd-audit-fix](gsd-audit-fix.md) | Autonomous audit-to-fix pipeline | High |
| [gsd-audit-uat](gsd-audit-uat.md) | Cross-phase UAT audit | Low |
| [gsd-audit-milestone](gsd-audit-milestone.md) | Full milestone audit | Medium |

## Debugging & Forensics

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-debug](gsd-debug.md) | Scientific method bug investigation | Medium–High |
| [gsd-forensics](gsd-forensics.md) | Post-mortem for failed workflows | Low–Medium |

## Brownfield / Codebase Analysis

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-map-codebase](gsd-map-codebase.md) | Parallel codebase analysis | High (one-time) |
| [gsd-graphify](gsd-graphify.md) | Build queryable knowledge graph | Medium |
| [gsd-intel-updater](gsd-intel-updater.md) | Update codebase intelligence index | Medium |

## Exploration & Ideation

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-explore](gsd-explore.md) | Socratic ideation before committing | Low |
| [gsd-sketch](gsd-sketch.md) | HTML mockup generation | Low–Medium |
| [gsd-spike](gsd-spike.md) | Feasibility experiments | Medium |

## Navigation & State

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-resume-work](gsd-resume-work.md) | Restore full context from prior session | Low |
| [gsd-pause-work](gsd-pause-work.md) | Create handoff for paused work | Low |
| [gsd-health](gsd-health.md) | Diagnose + repair .planning/ state | Low |
| [gsd-stats](gsd-stats.md) | Project statistics | Low |
| [gsd-update](gsd-update.md) | Update GSD to latest version | Low |
| [gsd-undo](gsd-undo.md) | Safe undo of last GSD operation | Low |

## Configuration & Workspace

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-settings](gsd-settings.md) | Interactive settings management | Low |
| [gsd-config](gsd-config.md) | Direct config manipulation | Low |
| [gsd-workspace](gsd-workspace.md) | Manage isolated workspace environments | Low |
| [gsd-workstreams](gsd-workstreams.md) | Manage parallel workstreams | Low |
| [gsd-thread](gsd-thread.md) | Persistent cross-session context threads | Low |

## Documentation & Capture

| Command | Purpose | Token Cost |
|---|---|---|
| [gsd-docs-update](gsd-docs-update.md) | Generate/update project documentation | Medium |
| [gsd-capture](gsd-capture.md) | Capture todos and backlog items | Low |
| [gsd-ingest-docs](gsd-ingest-docs.md) | Ingest external planning documents | Medium |
| [gsd-import](gsd-import.md) | Import external plans with conflict detection | Medium |
| [gsd-extract-learnings](gsd-extract-learnings.md) | Extract learnings from phase summaries | Low |
| [gsd-milestone-summary](gsd-milestone-summary.md) | Generate milestone summary for onboarding | Medium |

## Namespace Routers (token-efficient entry points)

| Router | Routes to |
|---|---|
| [gsd-ns-workflow](gsd-ns-workflow.md) | discuss, plan, execute, verify, phase, progress |
| [gsd-ns-project](gsd-ns-project.md) | milestones, audits, summary |
| [gsd-ns-review](gsd-ns-review.md) | code review, debug, security, eval, UI |
| [gsd-ns-context](gsd-ns-context.md) | map, graphify, docs, learnings |
| [gsd-ns-manage](gsd-ns-manage.md) | config, workspace, workstreams, thread, ship |
| [gsd-ns-ideate](gsd-ns-ideate.md) | explore, sketch, spike, spec, capture |

## Misc

| Command | Purpose |
|---|---|
| [gsd-autonomous](gsd-autonomous.md) | Fully autonomous execution mode |
| [gsd-quick](gsd-quick.md) | Ad-hoc task with GSD guarantees, no planning |
| [gsd-fast](gsd-fast.md) | Toggle fast mode (Opus with fast output) |
| [gsd-manager](gsd-manager.md) | Multi-phase dashboard |
| [gsd-review-backlog](gsd-review-backlog.md) | Promote backlog items to active milestone |
| [gsd-inbox](gsd-inbox.md) | Triage open GitHub issues/PRs |
| [gsd-profile-user](gsd-profile-user.md) | Generate developer behavioral profile |
| [gsd-pr-branch](gsd-pr-branch.md) | Create clean PR branch (filter .planning/ commits) |
| [gsd-cleanup](gsd-cleanup.md) | Archive completed phase directories |
| [gsd-plan-review-convergence](gsd-plan-review-convergence.md) | Cross-AI review of plans |
