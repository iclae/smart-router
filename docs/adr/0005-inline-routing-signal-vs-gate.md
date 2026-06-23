# Inline routing signal, distinct from the gate

The router has three outcomes — spawn, inline, gate — but only two were visible. A spawn shows a subagent; a gate shows an escalation prompt. The **inline** outcome was silent: the router sized the task, decided `Opus+medium` was right, and just proceeded. From the outside this is indistinguishable from the router never having fired, so a correct "inline, unchanged" decision looked like a no-op. We make every task boundary emit a one-line **routing signal** of its decision, whatever the outcome.

The apparent obstacle is the existing `don't-nag` rule (step 3). Always announcing seems to violate it. The resolution is that the rule and the signal target different objects:

- **Signal** — passive decision *log*, asks nothing of the user (`⟢ router: inline @ Opus+medium`).
- **Gate** — active escalation *prompt*, interrupts and asks the user to act (`consider /effort high`).

`don't-nag` governs the gate, never the signal. So always-on signalling is compatible with sparse gating. For a high-consequence, low-reversibility inline task — the one case that forfeits the executor/reviewer separation a spawn gets for free (ADR-0004) — the signal also carries the verifiable criterion the main will self-check against, at no extra cost since step 6 binds that criterion anyway.

## Considered Options

- **Fire only on non-trivial / changed decisions.** Rejected: it re-creates the exact blind spot being fixed — the user could not tell a silent trivial-inline from the router never running. The original complaint was precisely about the "nothing changed" case being invisible.
- **Peer-spawn for separation — a fourth outcome that spawns a same-profile subagent for high-stakes inline, purely to regain executor/reviewer separation.** Rejected. Inline tasks are by definition not off-profile, so the power-sizing benefit of a spawn is zero; only separation is on the table, and a same-profile spawn buys only *context* separation, not the *capability* separation ADR-0004 leans on (the reviewer is no smarter than the executor). It also hits the spawn floor (ADR-0003 — brief + cold-start cost exceeds the saving for small work) and forfeits the interactivity that exploratory inline work needs. The narrow high-stakes slice it could serve is already covered by binding a verifiable criterion and, when unverifiable, the human gate. A fourth outcome adds router cognitive load — the very thing the skill exists to save.

## Consequences

The decision space stays at three outcomes; the signal is reporting, not a new route. `don't-nag` is now explicitly scoped to the gate, which the glossary records under **Routing signal**. The residual confidently-wrong risk for inline work is unchanged from ADR-0004 — the signal makes the decision visible but does not restore the separation inline lacks; that remains the verifiable-criterion's job.
