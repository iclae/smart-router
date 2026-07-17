# Defending against confidently-wrong under-escalation

> **Status** Accepted · **Relates to** [ADR-0005](0005-inline-routing-signal-vs-gate.md), [ADR-0011](0011-gate-direction-only-read-from-externals.md) (the shape of layer 3's gate), [ADR-0012](0012-spawn-model-floor-tracks-the-reviewer-backstop.md) (what layers 1–2 holding, or not, costs at the spawn site)

A model that confidently believes it can do a task it actually can't produces no felt strain, so no introspective check (cheap-peek, round-up) can catch it. We defend this structurally, not by self-judgment, in three layers ordered by reliability.

1. **Bind the task to a verifiable criterion.** Self-assessment asks "what independent, cheaper check tells done-right from done-wrong?" — not "do I feel capable?". When such a check exists, a confidently-wrong result fails it regardless of confidence. This is domain-agnostic (re-derivation, check-against-source, consistency/constraint, reproducible/executable, checklist coverage), not coding-specific. Primary defense; dissolves the problem when the task is verifiable.

2. **Executor/reviewer separation for down-delegation.** The Opus main substantively reviews a weaker subagent's artifact against the criterion (not rubber-stamp). Free — main already integrates. Catches confident-wrong from below.

3. **Human gate for up-delegation / main-line, high-stakes + low-reversibility.** Main is the weaker party when delegating up and can't review a smarter result, and can't self-catch a main-line blind spot. The only backstop is surfacing the routing decision to the user — but only when the task is high-consequence and hard to reverse, to avoid nagging. This gate **fires predictively, not reactively**: at the first boundary where heavy work is foreseeable, never on felt strain. The confidently-wrong case produces no strain, so a reactive gate (one that waits for the model to feel the task is too hard) never fires for exactly the case layer 3 exists to catch — predictiveness is the load-bearing condition that makes the gate cover what it must.

## Consequences

Verifiability is itself a routing signal: a task with no nameable independent check, when high-stakes, routes to the human gate (layer 3). The posture shifts from "predict difficulty perfectly up front" (impossible against a blind spot) to "predict, then verify cheaply." A residual risk remains for tasks that are simultaneously unverifiable, up-delegated/main-line, and slip the stakes filter — this is the honest boundary of the design, not closable with more prompt.
