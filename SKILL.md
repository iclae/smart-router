---
name: smart-router
description: Right-size the model and effort per task — keep the main session as router and delegate off-profile work to subagents, adapting to the user-set baseline (model ∈ {Opus, Sonnet}, effort variable). Apply at every task boundary; a subagent reading this stands down (it is the delegatee, not the router).
---

A discipline the main session self-applies so heavy reasoning gets enough power and mechanical work doesn't burn it. The main session's baseline is a **variable** the user sets at session start — model ∈ {Opus, Sonnet}, effort whatever the user chose (ADR-0010) — and it plays router, never the heavy laborer. Vocabulary (Profile, Roster, Peak working set, Brief, Verifiable criterion, Yield) is defined in [`CONTEXT.md`](CONTEXT.md); the *why* behind each rule is in [`docs/adr/`](docs/adr/).

Run these at every **task boundary** — a new user request, or when you carve off a clearly self-contained, clearly off-profile sub-block. Never per tool call.

## 1. Yield if an orchestrator owns routing

If the current workflow is already run by a skill that self-routes model/effort/spawn (an orchestrator skill, whatever it's called in this setup), stand down — that skill owns routing. One orchestrator at a time. _Done when:_ you've confirmed no such skill is driving, or handed control to it.

## 2. Size the profile

Judge the task's needed **Profile** by introspection — how hard is the reasoning, how big is the working set — not by a fixed difficulty→model table. Map the felt difficulty onto the current **Roster** below for the model; pick effort separately (mechanical → low; deep deliberation → high/max). **Round up when unsure** — a wrongly-high profile wastes some tokens; a wrongly-low one produces a wrong result that costs far more to catch and redo — **but cap at the baseline**: this profile is the **subagent's** (applied only on down-delegation, step 3), and down-delegation never exceeds the baseline, so a task that rounds up past the baseline isn't spawnable-down — it stays inline or routes to the gate. The main session's baseline you cannot move (`/effort` is user-level), so a sub-baseline number here never means "drop the main to it." This effort is realized by selecting a subagent **type** pre-defined with it, not by passing it at the spawn call, which carries `model` only (ADR-0009). _Done when:_ you have a concrete (model, effort) for the task.

## 3. Spawn, do inline, or gate

Two independent forces pull a task off inline; **either** alone can justify a spawn:
- **Power** — the needed *model* differs from the baseline **downward** (down to a cheaper model for a separable chunk). Spawn is down-delegation only — it never raises the model above the baseline (a stronger subagent can't be reviewed by a weaker main, ADR-0004 layer 2; raising the main itself is a gate, below). Within the main session, "up" is two-dimensional — effort or model — but neither is self-settable (`/effort`, `/model` are user-level), so up is surfaced as a gate, not self-applied (ADR-0010).
- **Context economy** (ADR-0007) — the main needs only the *result*, not the process, and that process is verbose enough that keeping it inline would tax the rest of a long session.

Spawn when one force holds **and** both gates pass: the task is a self-contained sub-problem returning a compact artifact, **and** it clears the floor (for a context-economy spawn the floor is the *volume of process avoided*, not difficulty).

A task that doesn't spawn is not automatically inline — it is a **through-line** candidate, and before it's allowed to default to inline, check whether it needs a **human gate**: a task that needs more *power* than the baseline provides but can't be spawned off. This check is a mandatory precondition, not a third co-equal branch alongside spawn/inline: spawn-vs-inline answers *who executes*, the gate answers *whether the current power is enough* — the two are orthogonal, and a task can be through-line-bound for inline **and** still need a gate first.

You cannot raise your own session (`/effort`, `/model` are user-level), so surface it: "this may need higher power — consider raising effort or model." **Recommend the direction only** — not a target level, not which dimension: you cannot read your own runtime effort reliably, so either would be a claim you can't ground. The user picks (ADR-0011). Recommend more margin the higher the consequence and the lower the reversibility — a judgment, not a table.

Read "needs more power" off the task's **externals**, and fire **predictively** — at the first boundary where heavy work is foreseeable, carve-off mid-task only as a fallback — never on felt strain (ADR-0004). The externals: can you name a verifiable criterion (the strongest signal — its absence on a high-stakes task is itself a trigger, §6); consequence and reversibility; peak working set; shape (a long-horizon heavy piece?); and whether it's a through-line you can't spawn off. **Unsure → gate**: a needless gate costs one line, a missed one costs a confidently-wrong result. Reading the externals may take a bounded **probe** — read-only tools (Read/Grep/Glob) answering that checklist, stopping once it's answered; never "start it and see how it feels". Gate only high-consequence, low-reversibility tasks; don't nag.

Only once the gate check clears — or wasn't warranted — does the task run **inline**, including when the process is worth *retaining* as basis for later work: a spawn would discard the trail the main will need, so inline beats spawn there even if the task looks separable. Inline always runs at the user-set baseline (ADR-0010), whatever the effort sized in step 2 — you never lower the main session for a light task (lowering is a forward look at the whole session — below).

Don't proactively recommend *lowering*. Point that way only on a forward look — the foreseeable rest of the session no longer needs the current level — and when unsure, **don't lower** (ADR-0011).

Down-delegation calibration: once the hard part (e.g. the plan) is settled, execution drops in difficulty — spawn the separable chunk to the cheapest adequate model (selected per the Roster's current Pareto-efficient pairs, not a fixed name) and review its artifact (ADR-0004).

**Spawn model floor**: Haiku only when you can name a verifiable criterion *and* you will actually check the artifact against it — both, or the floor is Sonnet (ADR-0012). A review with no criterion is not a review. Unverifiable *and* high-stakes is not a floor question at all: gate it (§6).

Which subagent to spawn:

| Need | Type | Where it comes from |
|---|---|---|
| Down-delegation at a specific effort | `effort-low` / `effort-medium` / `effort-high`, with `model` passed at the spawn call | `agents/`, installed (ADR-0013) |
| Read-only exploration, cheaply | `explore-haiku` — **name it explicitly**; it can't shadow the built-in `Explore` (ADR-0013) | `agents/`, installed |
| Read-only exploration, inheriting the baseline model | built-in `Explore` | — |
| Peer-spawn for context economy, same profile | built-in `general-purpose` (inherits both) | — |
| Anything else where inherited effort is fine | built-in `general-purpose` | — |

**Then emit a one-line routing signal**, whatever the outcome — e.g. `⟢ router: inline @ <baseline>` (it fires at every task boundary, including "inline, unchanged"). For a high-consequence, low-reversibility task done *inline*, append the verifiable criterion you'll self-check against (ADR-0005, ADR-0004). For a **context-economy spawn** (ADR-0007 — spawned because the main needs only the result, not the verbose process), append a **falsifiable prediction** naming what the main will use going forward: `⟢ router: spawn (context economy) → main needs only [X], not the process`. It is falsified if the main later reaches back for something from the spawned-away process that `[X]` doesn't hold — a falsification surfaces a process-as-basis misjudgment, and the rate of it lower-bounds how often that entry judgment misfires (ADR-0016, [#13](https://github.com/iclae/smart-router/issues/13)). Power-delta spawns don't carry it — their "process not needed" is already settled by the separability precondition (ADR-0003). _Done when:_ the task is routed to inline, a spawn, or a gate, and a one-line signal of that decision has been emitted.

## 4. Apply the peak-working-set veto

Before finalizing the model, check the task's **Peak working set** — the most it must hold at one instant, not its total volume. If that may exceed the chosen model's window (per Roster), bump to the cheapest model whose window covers it and keep effort unchanged. Unknown working set → treat as large. High-volume but low-working-set tasks (rename across many files, run a suite) stay on the cheap model. _Done when:_ the model's window covers the peak, or you've bumped it.

## 5. Brief the subagent

A subagent is cold — it sees none of this session. Hand it the four-part **Brief**, load-bearing facts only:
1. **Objective + return shape** — the goal, and exactly what artifact to hand back.
2. **Load-bearing context** — what it can't re-derive: file paths, hard constraints, already-made decisions, and any methodology it must honor (it does *not* inherit the skill you're running, e.g. TDD). Pointers over pastes — paste only what you already derived at cost; point (path + function/section) to what the repo holds verbatim.
3. **Boundaries** — what's out of scope, what not to touch, when to stop.
4. **Return format** — "return only the artifact, not your process."

Parts 4 and the methodology half of part 2 are already fixed in the `effort-*` templates' bodies (ADR-0013) — briefing an installed type needn't restate them; briefing a built-in must.

_Done when:_ a competent stranger could do exactly this task, and nothing more, from the brief alone.

## 6. Verify the result

A confidently-wrong result has no felt strain, so introspection can't catch it — defend structurally (ADR-0004):
- **Bind a verifiable criterion.** Before trusting the result, name an independent, cheaper check that tells done-right from done-wrong (see menu below). A confidently-wrong result fails it.
- **Review down-delegated artifacts.** The main is (by definition) smarter than a *down*-delegated subagent — substantively check its artifact against the criterion, don't rubber-stamp. (This reviewer advantage holds only for true down-delegation; up-delegation loses it — see ADR-0004 layer 2.)
- **Gate the unverifiable.** If no independent check is nameable and the task is high-stakes, that absence is itself a routing signal — surface it to the user.

_Done when:_ the result passed a verifiable criterion, or was gated to the user.

### Verifiable-criterion menu (domain-agnostic)

A criterion is verifiable when checking it is *independent of* and cheaper than producing the work. Independence is what defeats the blind spot. Types:
- **Independent re-derivation** — reach the same answer by another path.
- **Check-against-source** — claims trace to the document, data, spec, or reality.
- **Consistency/constraint** — all stated constraints met, nothing self-contradictory, counts add up.
- **Reproducible/executable** — it runs, reproduces, resolves (tests and typechecks are the coding instances).
- **Checklist coverage** — every required element present, each item separately checkable.

## Roster (maintained — last checked 2026-07-17)

The one fact that goes stale on vendor changes. A *list*, not a difficulty→model mapping. Edit only this block when models change; the steps above never change. Update the date when you re-check.

| Tier (weak→strong) | Model id | Context window | Cost note |
|---|---|---|---|
| cheap-bulk | `claude-haiku-4-5-20251001` | 200K (64K max output) — verified 2026-07-17 | cheapest ($1/$5 per MTok) |
| solid-mid | `claude-sonnet-5` | native 1M | use full 1M for the veto |
| frontier | `claude-opus-4-8` | 1M (free on Max/Team/Enterprise; `/extra-usage` on Pro) | the higher end of the main-session range (ADR-0010); Opus and Sonnet 5 both valid baselines |
| apex | `claude-fable-5` | 1M (128K max output) — verified 2026-07-17 | $10/$50 per MTok, 2× Opus. **Not in the ADR-0010 baseline set** ({Opus, Sonnet}) — routing to it would be an ADR-0010 amendment, not a Roster edit. Quirks: thinking always on, raw reasoning never returned, safety classifiers may refuse |

- Effort scale: `low < medium < high < xhigh < max` (set per subagent via the `effort:` frontmatter field in the subagent definition file, **not** the spawn call — ADR-0009). `xhigh`: Fable/Mythos/Opus 4.7+/Sonnet 5 — not Haiku. `max`: not Haiku. Haiku 4.5 doesn't take the `effort` parameter at all.
- Pareto note (checked 2026-07-16, from a user-supplied capability/cost/token benchmark): `Sonnet+medium` was dominated by `Opus+low` on all three axes, and `Sonnet+high` by `Opus+high` — dominated pairs are not picks while this holds. A *note*, not a rule (ADR-0001): a snapshot of today's lineup, so re-check on any vendor change and correct it here only.
- `effort:` on a subagent is **documented to override session effort, and unverified** — the platform exposes no way to observe it (ADR-0013). Don't spend tokens trying to confirm a spawn's effort took; you can't.
