---
name: smart-router
description: Right-size the model and effort per task — run the main session at Opus+medium and delegate off-profile work to subagents.
disable-model-invocation: true
---

A discipline the main session self-applies so heavy reasoning gets enough power and mechanical work doesn't burn it. The main session stays **Opus+medium** and plays router, never the heavy laborer. Vocabulary (Profile, Roster, Peak working set, Brief, Verifiable criterion, Yield) is defined in [`CONTEXT.md`](CONTEXT.md); the *why* behind each rule is in [`docs/adr/`](docs/adr/).

Run these at every **task boundary** — a new user request, or when you carve off a clearly self-contained, clearly off-profile sub-block. Never per tool call.

## 1. Yield if an orchestrator owns routing

If the current workflow is already run by a skill that self-routes model/effort/spawn (e.g. `linear-task-worker`, `devflow`), stand down — that skill is the orchestrator. One orchestrator at a time. _Done when:_ you've confirmed no such skill is driving, or handed control to it.

## 2. Size the profile

Judge the task's needed **Profile** by introspection — how hard is the reasoning, how big is the working set — not by a fixed difficulty→model table. Map the felt difficulty onto the current **Roster** below for the model; pick effort separately (mechanical → low; deep deliberation → high/max). **Round up when unsure** — a wrongly-high profile wastes some tokens; a wrongly-low one produces a wrong result that costs far more to catch and redo. _Done when:_ you have a concrete (model, effort) for the task.

## 3. Spawn, do inline, or gate

Spawn a subagent only when **all three** hold:
- the profile differs from Opus+medium enough to matter (not medium↔high jitter), **and**
- the task is a self-contained sub-problem whose result returns as a compact artifact, **and**
- it's big enough that the brief+cold-start cost is worth it (the floor).

Otherwise do it inline. One exception needs a **human gate**: a hard task that needs more effort but is the main through-line (not spawnable) — you cannot raise your own session effort (`/effort` is user-level), so surface it: "this needs higher effort, consider /effort high." Gate only high-consequence, low-reversibility tasks; don't nag.

**Then emit a one-line routing signal**, whatever the outcome — e.g. `⟢ router: inline @ Opus+medium`. It's a passive decision log, not a prompt: the don't-nag rule above governs the *gate* (it asks you to act), never the signal (it only reports), so the signal fires at every task boundary even when the decision is "inline, unchanged" — the case that was previously invisible. When the outcome is a high-consequence, low-reversibility task done *inline* — the one case that forfeits the executor/reviewer separation a spawn gets for free (ADR-0004) — append the verifiable criterion you'll self-check against. _Done when:_ the task is routed to inline, a spawn, or a gate, and a one-line signal of that decision has been emitted.

## 4. Apply the peak-working-set veto

Before finalizing the model, check the task's **Peak working set** — the most it must hold at one instant, not its total volume. If that may exceed the chosen model's window (per Roster), bump to the cheapest model whose window covers it and keep effort unchanged. Unknown working set → treat as large. High-volume but low-working-set tasks (rename across many files, run a suite) stay on the cheap model. _Done when:_ the model's window covers the peak, or you've bumped it.

## 5. Brief the subagent

A subagent is cold — it sees none of this session. Hand it the four-part **Brief**, load-bearing facts only:
1. **Objective + return shape** — the goal, and exactly what artifact to hand back.
2. **Load-bearing context** — what it can't re-derive: file paths, hard constraints, already-made decisions, and any methodology it must honor (it does *not* inherit the skill you're running, e.g. TDD). Pointers over pastes — paste only what you already derived at cost; point (path + function/section) to what the repo holds verbatim.
3. **Boundaries** — what's out of scope, what not to touch, when to stop.
4. **Return format** — "return only the artifact, not your process."

_Done when:_ a competent stranger could do exactly this task, and nothing more, from the brief alone.

## 6. Verify the result

A confidently-wrong result has no felt strain, so introspection can't catch it — defend structurally (ADR-0004):
- **Bind a verifiable criterion.** Before trusting the result, name an independent, cheaper check that tells done-right from done-wrong (see menu below). A confidently-wrong result fails it.
- **Review down-delegated artifacts.** You (Opus) are smarter than a down-delegated subagent — substantively check its artifact against the criterion, don't rubber-stamp.
- **Gate the unverifiable.** If no independent check is nameable and the task is high-stakes, that absence is itself a routing signal — surface it to the user.

_Done when:_ the result passed a verifiable criterion, or was gated to the user.

### Verifiable-criterion menu (domain-agnostic)

A criterion is verifiable when checking it is *independent of* and cheaper than producing the work. Independence is what defeats the blind spot. Types:
- **Independent re-derivation** — reach the same answer by another path.
- **Check-against-source** — claims trace to the document, data, spec, or reality.
- **Consistency/constraint** — all stated constraints met, nothing self-contradictory, counts add up.
- **Reproducible/executable** — it runs, reproduces, resolves (tests and typechecks are the coding instances).
- **Checklist coverage** — every required element present, each item separately checkable.

## Roster (maintained — last checked 2026-06-23)

The one fact that goes stale on vendor changes. A *list*, not a difficulty→model mapping. Edit only this block when models change; the steps above never change. Update the date when you re-check.

| Tier (weak→strong) | Model id | Context window | Cost note |
|---|---|---|---|
| cheap-bulk | `claude-haiku-4-5-20251001` | verify before relying | cheapest |
| solid-mid | `claude-sonnet-4-6` | 1M-capable, **but Claude Code defaults to 200K — needs `/extra-usage`, which burns usage credits** | treat as 200K unless 1M confirmed for the session |
| frontier | `claude-opus-4-8` | 1M (free on Max/Team/Enterprise; `/extra-usage` on Pro) | main session lives here |

- Effort scale: `low < medium < high < max` (settable per subagent via frontmatter or spawn param).
- `claude-fable-5` exists but is unclassified here — verify its tier and window before routing to it.
