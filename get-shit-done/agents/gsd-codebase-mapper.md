# gsd-codebase-mapper

## Role
Analyzes an existing codebase and writes structured intelligence documents to `.planning/codebase/`. Runs in parallel with 3 sibling instances, each focused on a different analysis dimension. Used for brownfield onboarding and periodic re-indexing.

## Inputs
- Codebase files (read selectively based on focus area)
- Focus area assignment: `tech` | `arch` | `quality` | `concerns`

## Outputs
| Focus | Output File | Contents |
|---|---|---|
| `tech` | `.planning/codebase/tech.md` | Stack, dependencies, versions, build system |
| `arch` | `.planning/codebase/arch.md` | Architecture patterns, data flow, module boundaries |
| `quality` | `.planning/codebase/quality.md` | Test coverage, technical debt, code quality indicators |
| `concerns` | `.planning/codebase/concerns.md` | Security issues, performance hotspots, known problems |

## Lifecycle
1. Spawned by `map-codebase` orchestrator with a specific focus area
2. Reads relevant files (architecture-focused agent reads entry points, routers, config; quality-focused reads test files and CI config, etc.)
3. Produces structured analysis document
4. Returns — orchestrator waits for all 4 to complete before continuing

## Orchestration Trigger
`/gsd-map-codebase` command spawns 4 instances in parallel (one per focus area).

## Isolation
**Fresh 200K-token context** per instance. The 4 instances run simultaneously without sharing context.

**Tool access:** Read, Bash, Grep, Glob, Write

## Token Behavior
**High** — reads broadly across the codebase. Typical per-instance: 30K–100K tokens depending on codebase size. 4 parallel instances = 120K–400K total.

## Key Behaviors

### Read-Only Analysis
The codebase mapper never modifies code. It is purely analytical — reads files, produces documentation.

### Selective Reading
Each focus area reads strategically:
- **tech**: package.json/go.mod/Cargo.toml, build configs, Dockerfile
- **arch**: entry points, routers, service boundaries, data models
- **quality**: test files, coverage reports, lint configs, CI configs
- **concerns**: auth code, input parsing, external API calls, error handling
