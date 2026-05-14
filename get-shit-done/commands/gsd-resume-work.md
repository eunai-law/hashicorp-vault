# /gsd-resume-work

## Purpose
Restores full project context at the start of a new session. Reads STATE.md, ROADMAP.md, and phase artifacts to reconstruct exactly where work left off and what to do next.

## When to Use
- At the start of any session where GSD work is in progress
- After an interrupted execution (crash, manual cancel)
- When switching between projects

## Context Consumed
- `.planning/STATE.md` — stopped-at position, last decisions, blockers
- `.planning/ROADMAP.md` — phase completion status
- `.planning/phases/N/` — most recent phase artifacts

## Artifacts Created
None (read-only). Outputs a structured session briefing.

## Agents Spawned
None.

## Token Cost Profile
**Very Low** — reads artifacts, displays briefing. Under 3K tokens.

## Output
```
Session Briefing:
─────────────────
Phase: 2 (user-auth) — Status: Executing
Stopped: 2026-05-12 at plan-3 (2 of 3 tasks complete)
Last decision: "Use Redis for session cache (stateless JWT on client)"
Blockers: None

Recommended next step: /gsd-execute-phase 2 --plan 3 (resume plan-3)
```
