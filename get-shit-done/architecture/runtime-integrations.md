# GSD Runtime Integrations

## Supported Runtimes

GSD supports 14+ AI coding environments through a unified installation contract. The installer detects your runtime and deploys files with runtime-specific transformations.

| Runtime | Skill Prefix | Config Dir | Agent Spawning |
|---|---|---|---|
| Claude Code | `/gsd-` | `~/.claude/` | Full (Agent tool) |
| OpenCode | `/gsd:` | `~/.config/opencode/` | Full |
| Kilo | `/gsd:` | `./.kilo/` | Full |
| Gemini CLI | `/gsd:` | `~/.gemini/` | Via Gemini agents |
| Codex | `/gsd-` | `~/.codex/` | Partial |
| Cursor | `/gsd-` | IDE-specific | Partial |
| Windsurf | `/gsd-` | IDE-specific | Partial |
| Copilot | `/gsd-` | IDE-specific | Limited |
| Cline | `/gsd-` | IDE extension | Limited |

## Installer Logic

```bash
npx get-shit-done-cc@latest
```

The installer (`bin/install.js`) performs:

1. **Runtime detection** — Prompts for runtime selection or detects from environment
2. **Directory resolution** — Determines installation path (global `~/.claude/` vs local `.claude/`)
3. **File deployment** — Copies skill/command files with runtime-specific transformations
4. **Tool name mapping** — Translates Claude tool names to runtime equivalents:
   - Claude's `Bash` → Copilot's `execute`
   - Claude's `Read` → Copilot's `read_file`
5. **Hook translation** — Maps Claude Code hook event names to runtime equivalents
6. **Settings integration** — Merges required permissions into runtime settings files
7. **Path normalization** — Handles tilde expansion, Windows paths, container environments

### Container/Docker Fix

```bash
CLAUDE_CONFIG_DIR=/home/youruser/.claude npx get-shit-done-cc --global
```

Docker environments break tilde expansion. Setting `CLAUDE_CONFIG_DIR` explicitly before install prevents `~` from resolving to the wrong directory inside the container.

### Non-Interactive Install

```bash
# Install for Claude Code, globally, core profile only
npx get-shit-done-cc --runtime=claude --global --profile=core

# Install for Gemini CLI
npx get-shit-done-cc --runtime=gemini --global
```

All 15 runtimes have documented non-interactive flags. See docs/USER-GUIDE.md.

## Claude Code — Primary Runtime

Claude Code is the reference runtime. All GSD features work in Claude Code:

- Full agent spawning via the `Agent` tool
- Git worktree support for parallel workstreams
- Hook system (pre/post tool use events)
- Full MCP server integration
- Context7 library documentation lookup
- 200K token context windows per agent

Skills install to `~/.claude/skills/gsd-*/`. Each skill is a `.md` file read by Claude Code as a slash command definition.

**Settings merged into `~/.claude/settings.json`:**
```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(node:*)",
      "Bash(npm:*)",
      "Bash(gsd-sdk:*)"
    ]
  }
}
```

## Gemini CLI — Key Differences

- Slash command prefix uses `:` instead of `-`: `/gsd:plan-phase` not `/gsd-plan-phase`
- `resolve_model_ids: "omit"` set automatically — Gemini uses native model IDs, not Claude aliases
- Agent spawning works via Gemini's own agent mechanism, not Claude Code's `Agent` tool
- Some Claude-specific features (worktrees, certain MCP integrations) unavailable

## Non-Claude Runtimes — Common Limitations

When installing for non-Claude runtimes, the installer sets `resolve_model_ids: "omit"`, which instructs GSD to pass model selection to the runtime rather than translating Claude model IDs (opus, sonnet, haiku) to runtime-specific names.

Common capability gaps in non-Claude runtimes:
- **No git worktrees** — workstream isolation uses directory separation instead
- **Tool name mismatches** — installer translates most tool names; some advanced tools may be unavailable
- **No hook system** — runtime event hooks don't exist; hook-dependent features degrade gracefully
- **Context window differences** — not all runtimes offer 200K; GSD degrades to smaller context payloads

## Installer Idempotency

The installer is fully idempotent. Running it again:
- Overwrites existing GSD files with latest versions
- Does not duplicate settings entries
- Preserves user-modified config values where possible
- Safe to run on updates

## Profile System

Profiles control which skills are installed:

| Profile | Skills | Token overhead |
|---|---|---|
| `core` | 6 core-loop skills | Minimal |
| `standard` | core + phase management | Low |
| `full` (default) | All 40+ skills | Higher baseline |

Profiles compose: `--profile=core,audit` installs core skills plus audit tools.

The `/gsd:surface` runtime command enables/disables individual skill clusters after installation without reinstalling.

## Runtime-Specific Model Resolution

The 5-layer model resolution precedence:

1. Per-agent override in config.json
2. Dynamic routing tier escalation (adaptive profile)
3. Phase-type level settings (planning vs execution vs verification)
4. Global profile tier (quality / balanced / budget)
5. Runtime defaults

For Claude Code: tiers resolve to `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`
For non-Claude runtimes: tiers resolve to the runtime's native model IDs (or omitted if `resolve_model_ids: "omit"`)

## Skill Manifest Cache

```bash
gsd-sdk query skill-manifest
```

Pre-computes and caches skill discovery to avoid repeated filesystem scans at session start. The manifest maps all available GSD skills with their metadata. Used by hooks and the installer to enumerate capabilities without reading the full skills directory on every invocation.
