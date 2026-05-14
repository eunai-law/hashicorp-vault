# gsd-phase-researcher

## Role
Researches implementation approach for a specific phase before planning. Reads the codebase context, existing patterns, and (with web search) external documentation. Produces RESEARCH.md that feeds the planner.

## Inputs
- `CONTEXT.md` — locked decisions (research should validate, not contradict)
- `REQUIREMENTS.md` — phase requirements to research against
- `PROJECT.md` — project constraints
- Codebase files (existing patterns)
- Web search (for library docs, API references, best practices)

## Outputs
- `.planning/phases/N/RESEARCH.md` — implementation approach, risks, open questions, key decisions

## Lifecycle
1. Spawned by `plan-phase` orchestrator (parallel with gsd-pattern-mapper)
2. Reads CONTEXT.md and requirements
3. Explores codebase for relevant patterns
4. Searches web for implementation references (if needed)
5. Produces RESEARCH.md with structured findings
6. Returns — orchestrator waits for both researcher and pattern-mapper before spawning planner

## Orchestration Trigger
`plan-phase` orchestrator, parallel with `gsd-pattern-mapper`.

## Isolation
**Fresh 200K-token context.**

**Tool access:** Read, Write, Bash, Grep, Glob, WebSearch, WebFetch, mcp__context7__*, mcp__firecrawl__*, mcp__exa__*

## Token Behavior
**Medium–High** — reads codebase + potentially fetches external docs. Typical: 20K–50K tokens. Web-heavy research phases can reach 80K+.

## RESEARCH.md Structure
```markdown
# Phase N Research

## Recommended Approach
[Implementation strategy with rationale]

## Key Libraries/Patterns
[What to use and why]

## Risks
[Implementation risks and mitigations]

## Open Questions
[Anything CONTEXT.md didn't resolve]

## Reference Resources
[URLs, library docs, example implementations]
```
