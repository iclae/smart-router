# Main session is locked to Opus + medium

The main session always runs `Opus+medium` and offloads off-profile work to subagents, rather than switching its own model/effort per task.

## Considered Options

- **Cheaper orchestrator (e.g. Sonnet main).** Rejected on a hard constraint: the main session is the long-context accumulator (history + every returned artifact), so it needs the big window. Opus's 1M window is free on Max/Team/Enterprise; Sonnet's 1M always burns usage credits and is off by default in Claude Code. The role that needs the large window cheaply is Opus.
- **Always-high effort on main.** Rejected: main's real job is routing/brief-writing/integrating — meta-cognition, which is cheaper than solving. Heavy reasoning is spawned out to Opus-high subagents anyway. Always-high taxes the ~80% of light/routing turns, defeating the token goal.

## Consequences

"Up" for the main session means *effort only* (nothing is smarter than Opus). The main session cannot change its own session effort — `/effort` is user-level — so a hard task that is the main through-line (not a spawnable sub-problem) becomes a human gate: the router flags "consider /effort high", it does not self-escalate. The medium baseline's under-escalation risk has two shapes. The *uncertain* shape — the model feels strain — is covered by cheap-peek-for-strain-signals plus a lenient "round up when unsure" threshold, not by raising the baseline. The *confidently-wrong* shape — no felt strain, so introspection catches nothing — is not solvable by self-judgment and is handled structurally in ADR-0004.
