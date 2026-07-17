# The context-economy spawn signal carries a falsifiable prediction

> **Status** Accepted · **Relates to** [ADR-0007](0007-context-economy-spawn-driver.md) (the process-as-basis judgment this instruments), [ADR-0005](0005-inline-routing-signal-vs-gate.md) (the routing signal this extends), [ADR-0004](0004-defending-confidently-wrong-under-escalation.md) (the confidently-wrong blind spot that makes an external check necessary), [ADR-0017](0017-minimal-layer2-falsification-recheck.md) (the in-session recheck that catches this prediction's falsification)
> **Live rule** SKILL.md §3 (the routing-signal paragraph)

ADR-0007 admits context economy as a spawn driver: spawn when the main needs only the *result* and the process to reach it is verbose. Its counterweight is the *process-as-basis* judgment — keep it inline when the process is worth retaining. That entry judgment is a confidently-wrong-class call (ADR-0004): if the router wrongly judges a process to be result-only and spawns it away, nothing feels wrong, and the trail the main later needs is simply gone. [#13](https://github.com/iclae/smart-router/issues/13) hole 1 is exactly this exemption; hole 2 is that the project has no verification loop to tell how often it happens. You cannot gauge hole 1's severity by reasoning — it needs data.

This ADR builds the cheapest data-collection layer (layer 1 of #13's three-layer loop). On a context-economy spawn, the routing signal names a **falsifiable prediction**: *the main needs only `[X]` going forward, not the process*. The prediction is falsified by an external event — the main later reaching back for something in the spawned-away process that `[X]` doesn't hold — not by introspection. Counting the falsification rate gives a **lower bound** on how often the process-as-basis judgment misfires.

## Why context-economy spawns only

The prediction attaches to context-economy spawns, not all spawns. The process-as-basis judgment *is* the context-economy driver; a pure power-delta spawn settles "the process isn't needed" through its separability precondition (a self-contained sub-problem returning a compact artifact, ADR-0003), so the bet there is trivially true. Attaching the prediction to every spawn would dilute the falsification rate with a flood of trivial true-negatives from power-delta spawns, understating the misfire rate on the context-economy spawns that are the actual risk — and this is a *measurement* instrument, so dilution is a direct defect. The router already classifies the driver in §3, so restricting to context-economy spawns costs no extra step. (A spawn that is both power-delta *and* context-economy — a verbose exploration down-delegated to Haiku — qualifies as context-economy and carries the prediction.)

## Known limits, stated not hidden

- **Lower bound only.** A never-discovered miss — the main needed the process but neither the main nor the user ever realised it — is invisible to this layer. Catching those is layer 3's job (human review of the accumulated signal log). Layer 1 does not claim to find them; "no falsification" is not "no miss".
- **Still partly self-report.** The main noticing "I had to reach back" is itself introspection and shares the blind spot. But the falsification *event* — re-deriving or re-fetching what was spawned away — is more observable than the original judgment was, so it catches a fraction the entry judgment couldn't feel. That fraction is the point: a lower bound above zero is enough to trigger the hole-1 decision (#13).

## Considered Options

- **Attach to all spawns.** Rejected: dilution understates the rate the instrument exists to measure (above). This was a deliberate scope choice, not the issue's loose "spawn 的" phrasing taken literally.
- **No signal; have the main preview at spawn time whether the process will be needed.** Rejected: that is introspection, the exact thing ADR-0004 says cannot catch a confidently-wrong call. The design turns on an *external* falsification event, not a better prediction.
- **Skip layer 1; go straight to human log review (layer 3).** Rejected: layer 1 is near-free — one clause on a signal that already fires — and it produces the structured prediction that layer 3 later reviews. It is the substrate the expensive layer reads, not a redundant first pass.

## Consequences

This is the first piece of #13's verification loop actually built rather than only designed — the project moves from zero data to a lower-bound instrument. The routing signal's shape now varies by outcome: a high-stakes inline task carries its verifiable criterion (ADR-0005), a context-economy spawn carries this prediction. The *rate-counting* itself is not in SKILL.md — tallying falsifications across a session is analysis, not router action at a task boundary, so it stays with layer 3's log review; SKILL.md holds only the per-boundary emission (ADR-0008 cut-rule). Two residuals stay open and belong to #13, not here: never-discovered misses (need layer 3), and hole 1's own severity verdict (needs the rate this layer will produce).
