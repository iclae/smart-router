# The spawn model floor tracks the reviewer backstop

> **Status** Accepted · **Relates to** [ADR-0004](0004-defending-confidently-wrong-under-escalation.md) (the two layers this rule spends), [ADR-0010](0010-baseline-variable-not-locked-opus-high.md) (the main-session floor this mirrors)
> **Live rule** SKILL.md §3 (spawn model floor)

The main session's floor is Sonnet, never Haiku (ADR-0010). A spawned subagent's floor is Haiku — sometimes. This looks like an inconsistency and isn't: both floors are the same rule read at two sites. **The floor is Haiku exactly when a working executor/reviewer backstop stands behind the output, and Sonnet when none does.**

The main session never has one. Its output goes straight to the user, so there is no cheaper independent party between a confidently-wrong result and the person acting on it; ADR-0010 therefore fixes its floor at Sonnet unconditionally. A spawn usually *does* have one: ADR-0004 layer 2 gives down-delegation reviewer separation for free, because the main is by construction stronger than what it spawned down to and substantively checks the returned artifact.

But "usually" is not "always", and the backstop is not a property of spawning — it is a property of the **two ADR-0004 layers actually holding**:

- **Layer 1 must hold**: a verifiable criterion is nameable for this task.
- **Layer 2 must hold**: the main will actually check the artifact against that criterion, not wave it through.

Layer 2 depends on layer 1. A reviewer with no criterion is not a reviewer — it is a reader forming a second opinion with the same blind spot, which is the failure ADR-0004 exists to name. So when no criterion is nameable, *both* layers empty at once, the backstop disappears, and a Haiku spawn becomes structurally identical to the case ADR-0010 refuses for the main session: confidently-wrong output with nothing independent between it and the person who trusts it. The floor rises to Sonnet, meeting the main-session floor exactly where the reason for that floor also applies.

This is not a third rule bolted next to ADR-0010's. It is ADR-0010's rule with its condition made explicit.

## The rule

- **Haiku is a valid spawn target when** a verifiable criterion is nameable for the task **and** the main will review the returned artifact against it. Both, not either.
- **Otherwise the spawn floor is Sonnet** — the same floor the main session carries, for the same reason.
- **Unverifiable *and* high-stakes is not a floor question at all.** ADR-0004 layer 3 already routes it: the absence of a nameable check is itself the routing signal, and it goes to the gate rather than to any model. Raising the floor to Sonnet answers "which model" for the unverifiable work that is *low*-stakes enough to spawn at all; it does not license spawning unverifiable high-stakes work at Sonnet instead of gating it.

## Why review does not fully redeem Haiku

Haiku's characteristic failure on retrieval is **confident incompleteness** — stopping early, querying weakly, and reporting the result as settled. Reviewing the *artifact* cannot see this class of error: the main receives "no match found," which is indistinguishable from a true absence when the main cannot see the search that produced it. Layer 2 checks the answer, and the defect is in the question.

This is why the `explore-haiku` subagent returns the **search terms it used** alongside its findings: it converts part of an invisible process failure into a reviewable artifact, letting the main catch a wrong query even when it cannot catch a wrong conclusion. It is a partial fix — the main can now review the search *strategy*, but not whether the subagent stopped one directory short of the answer.

The remainder is accepted as **residual risk** on ADR-0004's terms: real, bounded by the criterion requirement above, and not closable with more prompt.

## Considered Options

- **Haiku is never a spawn target.** Rejected: it discards the cheapest model for exactly the work it is good at — high-volume, low-working-set, verifiable retrieval and transformation, where layer 1 is trivially satisfiable and layer 2 is free. The whole point of the router is that this work should not burn the baseline.
- **Haiku is always a spawn target; the review catches whatever goes wrong.** Rejected: it assumes layer 2 works unconditionally, when layer 2 is parasitic on layer 1. Without a nameable criterion the "review" is a rubber stamp, and the rule would license precisely the confidently-wrong path ADR-0004 was written against.
- **Make the floor a per-model list of allowed task types.** Rejected on ADR-0001 grounds: it encodes today's model weaknesses into the skill, so it goes silently wrong on the next vendor release. The criterion-and-review condition is a property of the *task and the routing*, not of Haiku, and survives Haiku being replaced.

## Consequences

The two floors stop being separate facts to remember: one condition (does a reviewer backstop actually stand behind this output?) explains both, and a reader who understands ADR-0010 already understands this. SKILL.md §3 carries the rule as action plus its calibration — Haiku needs a nameable criterion and a real review, otherwise Sonnet — and this ADR carries the derivation. The residual is Haiku's confident incompleteness on retrieval, partially mitigated by `explore-haiku`'s returned search terms, otherwise accepted.
