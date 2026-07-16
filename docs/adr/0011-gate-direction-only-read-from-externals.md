# The gate reads task externals and recommends a direction, not a target

> **Status** Accepted · **Relates to** [ADR-0004](0004-defending-confidently-wrong-under-escalation.md) (layer 3 — that the gate exists, and fires predictively), [ADR-0005](0005-inline-routing-signal-vs-gate.md) (gate vs signal; `don't-nag`), [ADR-0010](0010-baseline-variable-not-locked-opus-high.md) (the variable baseline that makes "up" two-dimensional), [ADR-0001](0001-judgment-to-model-roster-to-list.md) (why the magnitude recommendation is a judgment, not a table)
> **Live rule** SKILL.md §3 (the gate paragraph)

ADR-0004 established *that* the router needs a human gate for high-consequence through-line work it may not be up to, and that the gate must fire predictively rather than on felt strain. ADR-0010 made the main-session baseline a user-set variable, so "up" became two-dimensional — effort or model. Neither settles *how* the gate decides, or what it is allowed to claim when it fires. This ADR fixes four things: what the gate may say, how it reads "this needs more power", how much probing that read may cost, and when it points the other way.

## What the gate may say: a direction, never a target

The gate recommends **raising power** and stops there. It does not name an absolute target level ("go to `xhigh`"), because naming one asserts knowledge of the *current* level, and the router cannot obtain it. Three independent blind spots, verified 2026-07-15:

- The status-line JSON — the router's one structured view of its own session — **omits effort entirely** (Claude Code issue #36187).
- `max` and `ultracode` sessions run at an effort the router cannot see.
- A session-level override set after start, and a model-default hold, are both invisible to a config read.

Any one of them is enough. A router that says "raise to `high`" may be asking a session already at `max` for a *drop*, and would have no way to know it just did. So the gate emits the one thing it can ground in the task in front of it — the direction — and the **user**, who reads the current effort off the spinner, picks the destination. This is the router declining to pretend to a capability the platform does not provide, not a UX preference.

**Magnitude is advice, not arithmetic.** The gate may say how much margin the task warrants, and it sets that by risk: the higher the consequence and the lower the reversibility, the more headroom it recommends — insurance bought in proportion to the loss. This is a judgment the model makes at runtime from the task in front of it, deliberately *not* a risk→level mapping. ADR-0001's split applies unchanged: a table would bake today's lineup into the skill and go silently wrong on the next vendor change, whereas what actually moves — what each effort level buys — belongs in the Roster's effort note, the skill's one maintenance point for facts that go stale.

## Which dimension: the user's call, not the router's

The gate does not distinguish "raise effort" from "raise model". That call needs two things the router hasn't got:

1. **Runtime introspection.** "Does this need more thoroughness at the same ceiling, or a higher ceiling?" is unanswerable without knowing the current effort — exactly what the blind spots above hide.
2. **A reliable self-assessment of its own capability ceiling.** This is the class ADR-0004 rules out. A router that confidently concludes "my ceiling is the problem, not my thoroughness" is making precisely the confident, strain-free judgment ADR-0004 says cannot be trusted.

Both legs fail, so the dimension goes to the user along with the direction. The user holds what the router lacks: the spinner shows the current effort, and the choice costs them one keystroke.

## How the gate reads "this needs more power": externals, not strain

The gate cannot trigger on felt difficulty — ADR-0004's whole point is that the case it exists to catch produces none. So it reads the task's **externals**: properties visible before the work starts, independent of how the task feels.

- **Verifiability** — can you name an independent, cheaper check that tells done-right from done-wrong (the §6 menu)? The strongest signal, and the only one load-bearing on its own: its *absence* on a high-stakes task is already a gate trigger under ADR-0004 layer 3. It is strongest because it is the one reading that never routes through the router's self-image — the check either exists in the world or it doesn't.
- **Consequence and reversibility** — how bad is a wrong result, how hard to undo. This is ADR-0004's stakes filter, and (per the section above) the input that sets the recommended margin.
- **Peak working set** — §4's quantity, read here as a difficulty tell rather than as a window veto.
- **Shape** — a long-horizon heavy piece (sustained, multi-step, much held at once), as against a bounded mechanical one.
- **Prerequisite: not spawnable** — a through-line whose process the main must retain (ADR-0007) cannot be delegated away, and that is what leaves the gate as the only route. Separable heavy work is a spawn question, not a gate one.

**Round up toward incompetence.** When the read is unsure, gate. The asymmetry is the one §2's round-up already rests on, with a wider spread: a gate that need not have fired costs one line the user ignores, while a gate that should have fired and didn't delivers a confidently-wrong result on a high-consequence, hard-to-reverse task — the exact outcome the layer exists to prevent. We buy the cheap error to kill the expensive one: misses eliminated, over-fires accepted. `don't-nag` (ADR-0005) bounds the gate from the other side and does not conflict with this: what keeps the gate sparse is the stakes filter, not the confidence of the read.

## The probe: read-only, checklist-driven

Reading the externals may require looking at the repo, and the gate must fire *before* the work starts (ADR-0004 predictiveness) — so the look happens exactly when the router knows least. A bounded **probe** is allowed, under two constraints:

- **Read-only tools** (Read/Grep/Glob). A probe that overruns then wastes tokens but cannot quietly become the task.
- **Checklist-driven.** The probe answers the externals above — verifiability, consequence/reversibility, working set, shape — and stops the moment they are answered. It is not "start and see how it feels", which would re-import felt strain as the trigger *and* burn the baseline on the work the gate exists to move.

Both constraints are load-bearing together, neither alone: read-only still permits an unbounded read-only investigation that is, in cost, the task; the checklist alone still permits acting. Being unsure after a bounded probe is not grounds to keep probing — unsure is a gate.

## Pointing the other way: the down-shift is not the gate's mirror

ADR-0010 makes lowering available in shape (either dimension), but the router **does not proactively recommend it** as the gate's routine mirror. When it does point that way, the criterion is a forward look — *the foreseeable remainder of the session no longer needs the current level* — never a rear-view "the last task didn't need it", which would oscillate on every mechanical task inside a heavy stretch.

**When unsure, do not lower.** This round-up runs opposite to the gate's, for the same reason both are round-ups: the errors aren't symmetric. A needless hold costs tokens at the current level. A wrong lower runs the long-lived accumulator — which has no reviewer backstop of its own, its output going straight to the user — at reduced power, and that is ADR-0010's ground (B), still open.

We deliberately do **not** bind the down-shift to a verifiable criterion of its own, though ADR-0004 layer 1 nominally reaches it. The failure such a check would catch is oscillation (lower, then re-gate), whose cost is bounded and whose probability is already small once "unsure → don't lower" holds — while the check itself would run at every boundary. Its token cost exceeds the expected saving. The residual is a confidently-wrong "we won't need this again", accepted on ADR-0004's terms.

## Considered Options

- **Name an absolute target level ("raise to `xhigh`").** Rejected: it asserts the current level, which the three blind spots hide. Its appeal is that it is more actionable; its cost is being confidently wrong some unknown fraction of the time — including telling a `max` user to drop — with no way for the router to know which fraction.
- **Let the router pick the dimension (effort vs model).** Rejected on the two failed legs above. (First recorded in ADR-0010's option list, when the gate framework was still a forward reference; this ADR is its home.)
- **Trigger the gate on felt difficulty.** Rejected: the confidently-wrong case produces no strain (ADR-0004), so this never fires for exactly what the gate is for. Externals are the only reading independent of the blind spot.
- **A risk→level mapping for the magnitude recommendation.** Rejected on ADR-0001's grounds: it bakes the current lineup into the skill, goes stale on a vendor change, and duplicates what the Roster already maintains at one point.
- **Bind the down-shift to its own verifiable criterion.** Rejected on cost: a bounded, low-probability failure against a per-boundary check. Carried as residual instead.
- **No probe — judge from the request text alone.** Rejected: working set and verifiability often aren't readable from the request, so a router forced to guess them would round up to a gate on nearly everything — which `don't-nag` forbids (ADR-0005) and which turns the gate into noise.

## Consequences

The gate has a fixed shape now: it fires at the first boundary where heavy through-line work is foreseeable, on a read of the task's externals, unsure leaning to fire, optionally after a read-only checklist-bounded probe; it says "this may need higher power" with margin proportional to risk; and it names neither a target level nor a dimension. Both omissions are permanent — they follow from a platform fact (no effort introspection) and from ADR-0004 (no capability self-assessment), not from an unfinished design. A future proposal to make the gate more actionable by naming a level must first show the router can read its own effort. SKILL.md §3 carries the action and its calibration only; the derivations here are not duplicated there (ADR-0008). Two residuals stay open, neither closable with more prompt: a wrong down-shift's confidently-wrong "we won't need this again", and ADR-0004's own — the task that is unverifiable, on the through-line, and slips the stakes filter.
