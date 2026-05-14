# gsd-sketch

## Capability
Generates 2–3 HTML mockup variants for UI/design exploration before component implementation. Creates throwaway visual representations of layout options so you can evaluate and choose before writing production code.

## Orchestration vs Execution
**Execution-capable (standalone)** — writes HTML files directly. No agent spawning needed. The skill itself generates the mockups.

## How It Differs from Commands
`gsd-sketch` is a design exploration tool, not a workflow command. It does not produce planning artifacts consumed by other commands. It produces throwaway HTML files for visual evaluation.

Compare to `gsd-ui-phase` which produces a formal UI-SPEC.md design contract that planners must honor. Sketch comes before ui-phase — you explore visually, then lock decisions in the spec.

## Runtime Loading
Lightweight skill file. Installed globally, available in all sessions.

## Dependencies
None — can be run before any planning artifacts exist.

## Interactions
- Feeds into `/gsd-ui-phase` (choose a sketch direction, then formalize it)
- Feeds into `/gsd-discuss-phase` (sketch resolves UI ambiguity before locking decisions)

## Output
2–3 HTML files with different layout/UX approaches. Standalone files — open directly in a browser for comparison.

## When to Use
- "I need to decide between card layout vs list layout before I plan the dashboard"
- "I want to see what the checkout flow looks like before writing CONTEXT.md"
- "Show me 3 ways to handle the empty state for the search results page"

## Example
```
/gsd-sketch "User profile page with: avatar, bio, stats (followers, following, posts), 
recent posts grid, and follow button. Mobile-first."

# Produces:
# sketch-profile-v1.html  (minimal, text-focused)
# sketch-profile-v2.html  (card-heavy, visual)
# sketch-profile-v3.html  (sidebar layout)
```
