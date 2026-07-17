# Effort on spawn: set in the subagent file, not the spawn call

> **Status** Accepted · **Relates to** [ADR-0002](0002-main-session-locked-opus-high.md) (Superseded by [ADR-0010](0010-baseline-variable-not-locked-opus-high.md)), [ADR-0003](0003-spawn-boundary-and-orchestrator-yield.md), [ADR-0013](0013-ship-subagent-templates-not-bundled-agents.md) (how the definition-file channel this ADR names actually reaches a user)
> **Live rule** SKILL.md §2 (clarified), Roster effort note (corrected)

A subagent's **model** is settable at the spawn call (the Agent tool takes a `model` argument); its **effort** is not — the spawn call exposes no `effort` parameter. Effort is set instead in the **subagent definition file** (`frontmatter`'s `effort:` field), which overrides the session effort for the duration that subagent is active. The Claude Agent SDK confirms the split: `AgentDefinition` (the type a subagent file maps to) carries both `model` and `effort` (added in CHANGELOG #565, with `xhigh` in #914), while the spawn call carries only `model`.

So §2's effort sizing survives. The effort picked there is not a dead value: it is realized by spawning a subagent **pre-defined** with that effort, not by passing it at the spawn call. Down-delegation to `Sonnet+medium` means a subagent file declaring `model: sonnet, effort: medium` exists and is selected by `subagent_type`.

## Why two channels and not one

Two mechanisms serve different needs:

- **Subagent definition file** (`model:` + `effort:` in frontmatter) — the structural, durable channel. A fixed profile, written once, invoked many times by `subagent_type`. This is how a sized `(model, effort)` lands.
- **Spawn call** (`model` argument on the Agent tool) — the per-invocation, model-only channel. Useful when the model must vary per call but effort does not.

The Roster's older claim "settable per subagent via frontmatter **or spawn param**" conflated the two. The `effort` half is settable only through the first channel; the second carries `model` alone.

## Considered Options

- **Effort is inherited, not settable, on spawn (the earlier, rejected ADR-0009).** This conflated the two channels into one false claim that effort could not be set on a subagent at all. It collapsed because it would have dissolved §2's effort sizing — but the definition-file channel makes that sizing realizable. The error was treating "no effort on the spawn *call*" as "no effort on the subagent *at all*."
- **Allow effort to be named loosely as "configurable per subagent" without naming the channel.** Rejected: the channel matters for action. A router that sized `Sonnet+medium` and then tried to pass `effort: medium` at the spawn call would have it silently ignored — the actionable instruction is "select a subagent *type* pre-defined at medium," and that distinction must be visible at the step, not buried in an ADR.

## Consequences

§2's effort sizing stays, and now names its realization: the profile is realized by a subagent **type**, not a spawn **argument**. Down-delegation, the power trigger, the gate, and the inherited-by-default behavior when no effort is declared all stand as before — the correction is to *which channel* carries effort, not to whether effort is controllable. The Roster's effort note drops the false "spawn param" half and points to this ADR. Unchanged by ADR-0010's making the baseline variable: the main still cannot change its own effort — it only selects subagent types whose declared effort applies for the duration they are active.
