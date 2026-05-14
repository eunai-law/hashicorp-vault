# Skills vs Commands in GSD

## The Distinction

GSD uses both **commands** and **skills** as user-facing entry points, but they have different technical representations, loading mechanisms, and capabilities.

### Commands

Commands live in `commands/gsd/*.md` in the GSD repository. They are installed as **slash commands** into the AI runtime. In Claude Code, they appear as `/gsd-plan-phase`, `/gsd-execute-phase`, etc.

A command file is a structured Markdown document containing:
- YAML frontmatter with metadata
- XML-formatted prompt content
- References to planning artifacts via `@` includes
- Orchestration logic (what agents to spawn, in what order)

Commands are the **orchestrators** — they coordinate work but delegate implementation to agents.

### Skills

Skills live in `~/.claude/skills/gsd-*/` after installation (Claude Code) or equivalent runtime directories. In the GSD repo source, they are also under `commands/` but installed differently per runtime.

The term "skill" is used to describe **composable capabilities** that can be surfaced selectively. GSD's profile system installs skill subsets:

```bash
--profile=core      # 6 core-loop skills
--profile=standard  # core + phase management
--profile=full      # everything (default)
```

The `/gsd:surface` command enables or disables skill clusters at runtime without reinstalling.

## Technical Differences

| Dimension | Command | Skill |
|---|---|---|
| Location | `commands/gsd/*.md` | Installed to runtime skills dir |
| Loading | Per invocation (slash command) | Pre-loaded into runtime context |
| Context cost | Only when invoked | Contributes to baseline context |
| Capability | Orchestration + agent spawning | Orchestration or standalone |
| Scope | Full workflow | Focused capability |
| Profile control | No | Yes (`--profile`, `/gsd:surface`) |

## Namespace Routing Skills

The namespace skills (`gsd-ns-workflow`, `gsd-ns-project`, etc.) are a special category. They are **pure routers** — they present a compact menu of related commands and forward the user's intent to the appropriate concrete command.

They exist specifically to reduce baseline context usage. Six namespace skills at ~20 tokens each replace 86 individual command descriptions at ~25 tokens each.

```
Flat listing: 86 commands × 25 tokens ≈ 2,150 tokens always loaded
Namespace routing: 6 namespaces × 20 tokens ≈ 120 tokens always loaded
Savings: ~94% baseline reduction
```

## How Runtimes Load Skills

**Claude Code:**
- Skills are `.md` files in `~/.claude/skills/gsd-*/`
- Loaded into session context at startup
- Slash command prefix: `/gsd-`
- Tool names in agent frontmatter: `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `Agent`

**Gemini CLI:**
- Skills deployed to `~/.gemini/` equivalent
- Slash command prefix: `/gsd:` (colon, not hyphen — important difference)
- Tool names may differ; installer handles translation

**Codex / OpenCode / Kilo:**
- Similar pattern, runtime-specific directories
- Tool name mapping handled by installer (e.g., `Bash` → `execute`)
- `resolve_model_ids: "omit"` set automatically for non-Claude runtimes

**Cursor / Windsurf / Copilot:**
- Skills installed to IDE-specific directories
- Runtimes with limited agent-spawning capability may have reduced functionality

## What Can Spawn Agents

In Claude Code, both commands and skills can spawn agents using the `Agent` tool (when `Agent` is in their allowed tools list). The orchestrator pattern is the same whether the entry point is called a "command" or a "skill."

In runtimes without the `Agent` tool, spawning is unavailable. GSD degrades gracefully — some commands can still run sequentially without parallelization.

## When to Write a Skill vs a Command

GSD has a `write-a-skill` meta-skill for creating custom skills. The decision:

- **Write a skill** for: reusable capabilities you want available across sessions and projects; things you want to surface selectively; lightweight helpers
- **Write a command** for: full workflow orchestrations that involve multiple agent spawns, state management, and artifact production

## Lifecycle Differences

**Command lifecycle:**
1. User types `/gsd-execute-phase 1`
2. Claude Code reads the command file
3. Command prompt is added to session context
4. Command's orchestration logic runs in session
5. Agents are spawned (fresh contexts)
6. Session context accumulates orchestrator I/O

**Skill lifecycle:**
1. Skill descriptions are loaded at session start
2. User invokes skill (by name or namespace routing)
3. Skill logic runs
4. Session context accumulates skill I/O

The key difference: skills contribute to **baseline session context** from the start of a session, while commands only consume context when invoked.
