# /gsd-workstreams

## Purpose
Manages parallel workstreams — isolated `.planning/` contexts for concurrent development across independent areas of a project. Each workstream has its own phase state, preventing cross-contamination between parallel work tracks.

## When to Use
- When two independent feature areas need to be developed simultaneously
- When you have multiple developers working in parallel on the same project
- For large milestones with independent subsystems

## Context Consumed
- `.planning/config.json` — workstream configuration
- Per-workstream: `.planning/workstreams/{name}/` directories

## Artifacts Created
- `.planning/workstreams/{name}/` — isolated planning directory per workstream
- Per-workstream STATE.md, phases/, config.json

## Agents Spawned
None (management command). Execution within a workstream spawns agents normally.

## Token Cost Profile
**Low** — management operations only. Execution within workstreams has normal costs.

## Subcommands
```bash
/gsd-workstreams list                    # list active workstreams
/gsd-workstreams create --name api-v2   # create new workstream
/gsd-workstreams switch api-v2          # activate workstream
/gsd-workstreams status                 # show all workstream progress
/gsd-workstreams complete api-v2        # mark workstream complete
```

## Workstream Isolation
Each workstream gets:
- Its own `.planning/phases/` directory
- Its own `STATE.md`
- Its own git branch (optionally a worktree)

`gsd-sdk query init execute-phase <N> --ws <name>` routes all state reads/writes to the workstream directory.

## Example Workflow
```
/gsd-workstreams create --name auth-system
/gsd-workstreams create --name payment-integration

# Developer A:
/gsd-workstreams switch auth-system
/gsd-plan-phase 1
/gsd-execute-phase 1

# Developer B (simultaneously):
/gsd-workstreams switch payment-integration
/gsd-plan-phase 2
/gsd-execute-phase 2
```
