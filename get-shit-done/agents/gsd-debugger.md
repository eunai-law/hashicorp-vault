# gsd-debugger

## Role
Conducts a single cycle of scientific-method bug investigation. Identifies a hypothesis, gathers evidence, tests the hypothesis, and either identifies root cause or escalates with refined hypotheses for the next cycle.

## Inputs
- Bug description and reproduction steps
- Error messages, stack traces, logs
- Checkpoint from previous debug cycle (if continuation)
- Relevant codebase files

## Outputs
- Debug cycle findings (hypothesis tested, evidence gathered)
- Root cause identification (if found) + fix plan
- Refined hypotheses for next cycle (if not resolved)
- Checkpoint written to `.planning/debug/`

## Lifecycle
1. Receives payload from `gsd-debug-session-manager`
2. Forms or receives hypothesis from prior cycle
3. Reads relevant code, runs diagnostic commands
4. Tests hypothesis (confirms or refutes)
5. If confirmed: produces root cause + fix plan
6. If refuted: produces refined hypotheses + checkpoint
7. Returns to session manager

## Orchestration Trigger
Spawned by `gsd-debug-session-manager` for each investigation cycle.

## Isolation
**Fresh 200K-token context** per cycle. State carried between cycles via `.planning/debug/checkpoint.md` artifacts, not context inheritance.

**Tool access:** Read, Write, Edit, Bash, Grep, Glob, WebSearch

## Token Behavior
**Medium** — reads code files, runs diagnostic commands. Typical: 15K–40K tokens per cycle. Complex bugs may require 3–5 cycles.

## Scientific Method Protocol
1. **Observe**: Read error, stack trace, reproduce if possible
2. **Hypothesize**: Form specific, testable hypothesis ("The issue is X because Y")
3. **Test**: Gather evidence for/against hypothesis (read code, add logging, run commands)
4. **Conclude**: Confirm root cause or refute and refine hypothesis
5. **Fix**: If root cause found, produce minimal fix plan
