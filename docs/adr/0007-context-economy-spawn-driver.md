# Context economy is a co-equal spawn/inline driver

> **Status** Accepted · **Amends** [ADR-0003](0003-spawn-boundary-and-orchestrator-yield.md) · **Relates to** [ADR-0005](0005-inline-routing-signal-vs-gate.md), [ADR-0016](0016-context-economy-spawn-carries-falsifiable-prediction.md) (instruments this ADR's process-as-basis judgment with a falsifiable prediction)

The original spawn rule (ADR-0003) gated on a single driver: the needed *profile* differs from the baseline enough to matter. That misses a second, independent reason to move a task off inline — what the main session has to *carry* afterward.

A task's process can be either a cost or an asset to the main context:

- **Result-only, verbose process → spawn (context economy).** When the main needs only the *result* and the process to reach it is bulky (long tool outputs, exploration, log-grinding), doing it inline leaves that process squatting in the main context for the rest of a long session, taxing every later turn. A spawn keeps the process in the subagent and returns only the compact artifact — even when the profile is unchanged (`Opus+high` doing `Opus+high` work). The floor still applies, but here it is measured in *volume of process avoided*, not difficulty.
- **Process-as-basis → inline (context retention).** When the process is worth *retaining* as background or justification for later work, a spawn would discard the trail the main will need. Here inline beats spawn even if the task looks separable.

So the spawn condition becomes: *(power delta **or** context economy) **and** separable **and** clears the floor*. Context economy is not a fourth outcome — the outcomes stay inline / spawn / gate — only an additional input to the existing inline↔spawn decision.

## Reconciliation with ADR-0005

ADR-0005 rejected a same-profile "peer-spawn." That rejection stands, but it was scoped to one motivation: regaining **executor/reviewer separation**, which a same-profile spawn can't provide (the reviewer is no smarter than the executor — *capability* separation). Context economy wants a different thing from the same mechanism: **context** separation — keeping a verbose, result-only process out of the long main session. ADR-0005 itself noted a same-profile spawn *does* buy context separation; it simply wasn't what that ADR needed. So this is a complement, not a contradiction: peer-spawn for reviewer separation stays rejected; peer-spawn for context economy is admitted.

## Consequences

The router carries slightly more per-decision load — it now also asks "does the main need this process later?" alongside "is this off-profile?". That cost is accepted because it directly serves the token-economy goal the skill exists for (a long main session staying lean). The residual risk is over-spawning for hygiene on small tasks; the volume-based floor guards against it.
