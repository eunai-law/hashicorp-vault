# GSD Agents Index

All 31 specialized agents in the GSD roster. Every agent starts with a fresh 200K-token context — none inherit conversation history from the orchestrator.

## Execution Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-executor](gsd-executor.md) | Implements plan tasks, creates atomic commits | execute-phase orchestrator | Fresh — reads PLAN.md + codebase |
| [gsd-verifier](gsd-verifier.md) | Goal-backward phase verification | verify-work orchestrator | Fresh — reads plans + summaries + requirements |

## Planning Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-planner](gsd-planner.md) | Creates PLAN.md files from research | plan-phase orchestrator | Fresh — reads RESEARCH.md + CONTEXT.md |
| [gsd-plan-checker](gsd-plan-checker.md) | Validates plans against 8 dimensions | plan-phase orchestrator | Fresh — reads all PLAN.md files |
| [gsd-roadmapper](gsd-roadmapper.md) | Creates ROADMAP.md | new-project orchestrator | Fresh — reads research synthesis |

## Research Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-phase-researcher](gsd-phase-researcher.md) | Phase implementation research | plan-phase orchestrator | Fresh |
| [gsd-pattern-mapper](gsd-pattern-mapper.md) | Maps new files to existing codebase analogs | plan-phase orchestrator | Fresh |
| [gsd-project-researcher](gsd-project-researcher.md) | Domain/competitive/tech research | new-project orchestrator | Fresh |
| [gsd-research-synthesizer](gsd-research-synthesizer.md) | Combines parallel researcher outputs | new-project orchestrator | Fresh |
| [gsd-advisor-researcher](gsd-advisor-researcher.md) | Gray-area decision analysis | discuss-phase (on demand) | Fresh |
| [gsd-assumptions-analyzer](gsd-assumptions-analyzer.md) | Analyzes codebase assumptions | discuss-phase | Fresh |

## Checker / Validator Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-integration-checker](gsd-integration-checker.md) | Cross-phase integration verification | validate-phase | Fresh |
| [gsd-nyquist-auditor](gsd-nyquist-auditor.md) | Test coverage mapping to requirements | plan-phase | Fresh |
| [gsd-ui-checker](gsd-ui-checker.md) | Validates UI-SPEC.md | ui-phase | Fresh |
| [gsd-plan-checker](gsd-plan-checker.md) | 8-dimension plan validation | plan-phase | Fresh |

## Debug Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-debug-session-manager](gsd-debug-session-manager.md) | Manages multi-cycle debug loop | debug command | Fresh per cycle |
| [gsd-debugger](gsd-debugger.md) | Individual debug investigation cycle | debug-session-manager | Fresh |

## Code Quality Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-code-reviewer](gsd-code-reviewer.md) | Reviews source files for bugs/security/quality | code-review command, ship | Fresh |
| [gsd-code-fixer](gsd-code-fixer.md) | Applies fixes from REVIEW.md | code-review --fix | Fresh |
| [gsd-security-auditor](gsd-security-auditor.md) | Verifies threat model mitigations | secure-phase | Fresh |

## UI Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-ui-researcher](gsd-ui-researcher.md) | Produces UI-SPEC.md design contract | ui-phase | Fresh |
| [gsd-ui-auditor](gsd-ui-auditor.md) | Retroactive 6-pillar visual audit | ui-review | Fresh |

## AI Integration Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-ai-researcher](gsd-ai-researcher.md) | AI framework research for AI phases | ai-integration-phase | Fresh |
| [gsd-domain-researcher](gsd-domain-researcher.md) | Business domain research for AI systems | ai-integration-phase | Fresh |
| [gsd-eval-planner](gsd-eval-planner.md) | Designs AI evaluation strategy | ai-integration-phase | Fresh |
| [gsd-eval-auditor](gsd-eval-auditor.md) | Audits AI evaluation coverage | eval-review | Fresh |
| [gsd-framework-selector](gsd-framework-selector.md) | Selects AI framework via decision matrix | ai-integration-phase | Fresh |

## Codebase Intelligence Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-codebase-mapper](gsd-codebase-mapper.md) | Parallel codebase analysis (4 focus areas) | map-codebase | Fresh |
| [gsd-intel-updater](gsd-intel-updater.md) | Updates queryable intelligence index | map-codebase | Fresh |

## Documentation Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-doc-writer](gsd-doc-writer.md) | Writes/updates project documentation | docs-update | Fresh |
| [gsd-doc-classifier](gsd-doc-classifier.md) | Classifies planning documents | ingest-docs | Fresh (parallel) |
| [gsd-doc-synthesizer](gsd-doc-synthesizer.md) | Synthesizes ingested documents | ingest-docs | Fresh |
| [gsd-doc-verifier](gsd-doc-verifier.md) | Verifies doc claims against codebase | docs-update | Fresh |

## Profiling Agents

| Agent | Role | Spawned By | Context |
|---|---|---|---|
| [gsd-user-profiler](gsd-user-profiler.md) | Behavioral profile from session analysis | profile-user | Fresh |
