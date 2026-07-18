# Minimal layer 2: an in-session falsification recheck closes the prediction loop

> **Status** Accepted · **Relates to** [ADR-0016](0016-context-economy-spawn-carries-falsifiable-prediction.md) (the layer-1 prediction this catches), [ADR-0007](0007-context-economy-spawn-driver.md) (the process-as-basis judgment ultimately measured)
> **Live rule** SKILL.md §7

ADR-0016 has a context-economy spawn emit a falsifiable prediction. But emission alone is **inert**: the prediction sits in a session transcript and evaporates when the session ends. Nothing catches a falsification, so layer 1 produces no data — pure ceremony. To make it yield anything, a falsification has to be detected and recorded.

[#13](https://github.com/iclae/smart-router/issues/13) filed that detection under layer 2 (in-session recheck) and layer 3 (human log review), both deferred "pending data." That created a deadlock: layer 1 produces no data without at least minimal detection, yet detection was being deferred until there was data. This ADR breaks it by building the **minimal** detection that unblocks the data — not the full layer-2 defense, only the catch-and-record layer 1 needs to stop being inert.

## Mechanism: detect and record at the moment

- **Detect.** Reaching back into a process spawned away under a context-economy prediction — re-deriving or re-fetching what the returned `[X]` didn't hold — falsifies that prediction. This is caught at the moment because a reach-back is a *salient* event (you are visibly redoing work the artifact was supposed to have made unnecessary), not a speculative scan the router runs at every boundary against nothing.
- **Record — at the detection moment, not at session end.** The falsification is recorded to #13 the instant it is detected. The tempting design was to batch the session's falsifications and record them at wrap-up, but the model has no reliable signal for "the session is ending": it runs per user turn and never sees a last-turn marker — the same blind spot ADR-0015 built the model-invocability fix around. A wrap-up-triggered record would depend on an unobservable event and could simply never fire. The detection moment, by contrast, *is* observable, and is itself a task boundary — so anchoring the record there makes it reliably executable and keeps §7 within the per-boundary cadence rather than an exception to it. Falsifications are rare (a common one would already have surfaced hole 1), so per-detection recording does not flood #13, and the batching a wrap-up would have bought is not worth a dependency on an event the model can't see.

## This is measurement, not the deferred defense

Layer 2 as #13 designed it was a *defensive* recheck: catch the misjudgment and correct the routing. This builds only the **measurement** half — detect and record for data — and deliberately no corrective action: a falsification is logged, not routed around. Whether a real defense is worth building (or the process-as-basis calibration in §3 should tighten) still waits on the rate this produces, i.e. hole 1's severity (#13). So this does not pre-empt the deferred decision; it supplies the data that decision was waiting on.

## Limits, stated

Only in-session falsifications are caught. A never-discovered miss, or one that first bites in a *later* session, stays invisible — that is layer 3's job (human review of the accumulated signal log). This layer is therefore a lower bound on ADR-0016's already-lower bound; it is honestly the cheapest thing that produces a non-zero signal, not a complete count.

## Considered Options

- **Batch and record at session wrap-up.** Rejected: the model cannot observe "session end" — it runs per turn with no last-turn signal (ADR-0015's blind spot), so a wrap-up-triggered record depends on an unobservable event and may never fire. Recording at the observable detection moment is the executable form; the batching a wrap-up would buy is not worth the dependency. (This also disposes of the weaker variant — a pure wrap-up *sweep* that also defers detection — which additionally relies on end-of-session recall the detection moment already holds.)
- **Persist signals via a hook or log file for offline aggregation.** Rejected for now on ADR-0013's grounds (machinery outside the skill, a second install point, drift). The manual record-to-#13 is the cheapest thing that produces data; a scraper or hook is a layer-3 upgrade to take once volume justifies it.
- **Leave layer 1 emit-only; build nothing.** Rejected: emit-only produces no feedback, making the layer-1 prediction pure ceremony. This option is what exposed the gap — the loop was open — and doing nothing keeps it open.

## Consequences

Layer 1 now yields data: in-session falsifications reach #13, and the loop is closed for that case. Cross-session and never-discovered misses remain open, explicitly layer 3's. SKILL.md gains §7, whose detect-and-record both fire at the task boundary where the reach-back occurs — so it needs no session-end trigger and is no exception to the "run at every task boundary" cadence. The recording destination (#13) is a dogfood-phase convention — the whole falsifiable-prediction instrument is dogfood (ADR-0016), not a permanent general-skill feature — and generalizes to "the project's verification-loop tracker" if smart-router is ever used elsewhere. When enough rate data accumulates, the hole-1 severity call — and with it whether to build a real layer-2 defense or a layer-3 scraper — is a future session's, fed by what this collects.
