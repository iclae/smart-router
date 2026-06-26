# Main session is locked to Opus + high

> **Status** Accepted · **Relates to** [ADR-0004](0004-defending-confidently-wrong-under-escalation.md), [ADR-0006](0006-switching-cost-shapes-the-gate.md)

The main session always runs `Opus+high` and offloads off-profile work to subagents, rather than switching its own model/effort per task.

`high` — not the earlier `medium` — because the official effort guidance for Opus 4.8 makes `high` the default on every surface (Claude Code included) and tells coding/agentic users to *start* at `xhigh`, stepping down to `medium` only after measuring that quality holds on their evals. The main session is exactly an agentic-coding surface, and we had no such measurement: locking it below the default was an un-measured downgrade the docs caution against.

## Considered Options

- **Cheaper orchestrator (e.g. Sonnet main).** Rejected on a hard constraint: the main session is the long-context accumulator (history + every returned artifact), so it needs the big window. Opus's 1M window is free on Max/Team/Enterprise; Sonnet's 1M always burns usage credits and is off by default in Claude Code. The role that needs the large window cheaply is Opus.
- **Keep `medium` to spare the light turns.** This was the original choice, rejected on re-examination. Its premise — that `high` taxes the ~80% of light routing/integration turns — assumed effort is a fixed token budget. The docs say it is a *behavioral signal*: at the same effort level Claude thinks *less* on easy problems and more on hard ones, and at lower effort it may skip thinking on simple problems but also thinks less on the hard ones. So `high` costs little on a trivial routing turn (the problem is easy regardless), while `medium` underthinks exactly the hard turns where the confidently-wrong shape hides (ADR-0004). The tax `medium` bought is small; the risk it ran is not.
- **Always-`xhigh` on main.** Rejected: the docs scope `xhigh` to long-horizon (>30 min, million-token) agentic runs. The main's standing job is routing/brief-writing/integrating — meta-cognition, lighter than solving — and heavy long-horizon reasoning is spawned out to subagents anyway. `xhigh` is the gate's escalation target for a hard inline through-line, not the baseline.

## Consequences

"Up" for the main session means *effort only* (nothing is smarter than Opus). The main session cannot change its own session effort — `/effort` is user-level — so a hard task that is the main through-line (not a spawnable sub-problem) becomes a human gate: the router flags "consider /effort xhigh", it does not self-escalate. That gate is not free: switching effort mid-session re-reads the whole accumulated history (ADR-0006), a third independent reason — alongside the big-window and cheap-meta-cognition arguments above — to lock the profile rather than fiddle with it per task. The medium→high move shrinks but does not erase the under-escalation risk; its two shapes are unchanged. The *uncertain* shape (felt strain) is covered by round-up-when-unsure; the *confidently-wrong* shape (no felt strain) is not solvable by self-judgment and is handled structurally in ADR-0004.
