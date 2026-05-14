# /gsd-health

## Purpose
Diagnoses `.planning/` directory health, detects corruption or drift, and optionally repairs issues. Also provides context-window utilization monitoring via `--context`.

## When to Use
- When GSD commands behave unexpectedly
- After manual edits to `.planning/` files
- When STATE.md seems out of sync with the actual codebase
- Periodically on long projects as preventive maintenance

## Context Consumed
- All `.planning/` files
- git log (for commit/phase sync checking)

## Artifacts Created
- Diagnostic report (stdout)
- Repaired files (with `--repair`)

## Agents Spawned
None — uses `gsd-tools.cjs validate health`.

## Token Cost Profile
**Very Low** — local file validation, no AI agents. Under 1K tokens.

## Flags
```bash
/gsd-health              # diagnose
/gsd-health --repair     # diagnose + auto-repair
/gsd-health --context    # show context window utilization
```

## Context Window Monitoring
`--context` emits utilization percentage and status:
- `ok` — below 60%
- `warn` — 60–70% (consider `/clear` between commands)
- `critical` — above 70% (quality degradation risk; use `/clear` immediately)

## Common Issues Detected
- Phase numbering gaps (missing phase directories)
- STATE.md/ROADMAP.md inconsistency
- Missing SUMMARY.md files for completed plans
- Corrupt frontmatter in planning files
- Broken `@` references in plan files
