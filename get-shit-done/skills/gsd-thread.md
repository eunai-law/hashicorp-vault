# gsd-thread

## Capability
Manages persistent context threads — named, cross-session context artifacts that carry information about a specific topic (a design decision, a research finding, an ongoing investigation) across multiple sessions without relying on conversation history.

## Orchestration vs Execution
**Standalone** — manages artifact files. No agents spawned.

## How It Differs from Commands
Commands drive workflow execution. `gsd-thread` is a context persistence tool — it solves the problem of needing to carry specific information across sessions for work that spans multiple days or context resets.

Unlike STATE.md (which tracks project-wide position), threads track topic-specific context at a finer granularity.

## Runtime Loading
Lightweight skill file. Global install.

## Dependencies
Writes to `.planning/threads/` directory.

## Interactions
- Used with `/gsd-resume-work` to restore thread context at session start
- Can be referenced in CONTEXT.md ("see thread: auth-decision-log")
- `/gsd-pause-work` can include thread references in the handoff

## Subcommands
```bash
/gsd-thread create auth-tradeoffs    # create new thread
/gsd-thread append auth-tradeoffs "Decided against OAuth for v1 — too complex for single-tenant"
/gsd-thread read auth-tradeoffs      # display thread content
/gsd-thread list                     # list active threads
/gsd-thread close auth-tradeoffs     # archive completed thread
```

## When to Use
- Long-running design decisions that need a running log
- Research findings you want to reference across sessions
- Ongoing investigations that span multiple work sessions
- Notes about technical debt items being actively tracked
