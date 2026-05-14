# /gsd-autonomous

## Purpose
Enables fully autonomous execution mode — GSD runs the complete discuss → plan → execute → verify loop for a phase without pausing for confirmation. Intended for use with `claude --dangerously-skip-permissions`.

## When to Use
- When you want to walk away and return to completed work
- For well-understood phases where you trust the CONTEXT.md decisions
- In CI/CD contexts (experimental)

## Context Consumed
- All standard planning artifacts for the target phase
- `.planning/config.json` — must have `mode: "yolo"` or `--yolo` flag

## Token Cost Profile
**Very High** — runs the full loop: discuss (auto) + plan + execute + verify. All standard costs apply.

## Risks
- No human checkpoints. If the planner makes a wrong assumption, it executes the wrong thing.
- Requires clean git state — autonomous mode commits aggressively.
- Not recommended for phases that touch critical shared infrastructure.

## Flags
```bash
/gsd-autonomous 1              # autonomous execution of phase 1
/gsd-autonomous 1 --yolo       # skip all confirmation prompts
/gsd-autonomous --next         # autonomously run next pending phase
```
