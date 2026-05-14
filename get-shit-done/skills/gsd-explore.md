# gsd-explore

## Capability
Socratic ideation and idea routing. Helps you think through ideas, approaches, and architecture choices before committing to plans. Uses structured questioning to surface assumptions, trade-offs, and hidden constraints.

## Orchestration vs Execution
**Standalone** — no agents spawned. Pure conversation between you and the skill. This is the cheapest entry point into GSD for exploratory thinking.

## How It Differs from Commands
Commands produce artifacts and drive execution. `gsd-explore` produces clarity in your thinking. It does not write PLAN.md files, spawn executors, or create commits. Its output is your improved understanding.

Think of it as a pre-discuss conversation: before you run `/gsd-discuss-phase`, you might `/gsd-explore` to figure out what questions you need to answer.

## Runtime Loading
Installed as a lightweight skill file. In Claude Code: `~/.claude/skills/gsd-explore.md`. Available at all times without invocation cost.

## Dependencies
None — no planning artifacts required.

## Interactions
- Feeds into `/gsd-discuss-phase` (you explore, then lock decisions)
- Feeds into `/gsd-spike` (you explore, then run experiments)
- Feeds into `/gsd-sketch` (you explore UI concepts, then generate mockups)

## When to Use
- "I'm not sure which approach to take for this feature"
- "I want to think through the trade-offs before planning"
- "I have a rough idea but it needs refinement before I can write CONTEXT.md"

## Example
```
/gsd-explore "I need to add real-time notifications. 
Considering WebSockets vs SSE vs polling. 
Our infra uses Kubernetes, Redis, and Postgres."

# GSD asks structured questions:
# - How many concurrent users?
# - One-way (server → client) or bidirectional?
# - Message ordering guarantees needed?
# - Reconnection handling priority?
```
