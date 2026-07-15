# Main-session baseline is variable, not locked to Opus+high

> **Status** Accepted · **Supersedes** [ADR-0002](0002-main-session-locked-opus-high.md)
> **Live rule** SKILL.md §2 (baseline), §3 (up-gate is now model-or-effort)

ADR-0002 fixed the main session at `Opus+high`. Two facts that held up that lock have since moved, and the gate framework (ADR-0011, the effort/model-dimension handling) structurally weakens the third — emptying one of its two grounds, demoting the other to residual. The main session's baseline is now a **variable** the user sets at session start, within bounds; the router adapts to it rather than assuming it.

## What changed

1. **The window argument is void.** ADR-0002's load-bearing reason to prefer Opus for the main session was the large context window: Opus's 1M window was free on Max/Team/Enterprise, while Sonnet's 1M burned usage credits and was off by default in Claude Code. As of Sonnet 5, **Sonnet 5 always runs with the 1M window — no 200K variant, no beta header, no usage-credit cost on any plan** (official model-config docs, verified 2026-07-16). "The main needs the big window" no longer implies "the main needs Opus" — Sonnet 5's 1M satisfies it. (Sonnet 5's status as the *default* model varies by account tier — Pro/Team-Standard seats default to Sonnet 5, Max/Team-Premium/API to Opus 4.8 — but this is irrelevant to the window argument: whichever a user lands on, both carry 1M, so neither is excluded from the main session for lack of window.)

2. **The effort assumption was always user-overridable.** ADR-0002 treated `high` as the baseline effort, leaning on "Opus 4.8 defaults to high." But `/effort` is a user-level knob the router cannot move; a user may open a session at `Opus+medium` or `Sonnet+high`. Assuming the baseline is a fixed value the router didn't set was an overreach. The router now treats the effort it finds at session start as the baseline, whatever it is.

3. **The ratchet asymmetry that defended against lowering is structurally weakened, not fully emptied, by the gate framework.** ADR-0002 (and the closed investigation in issue #3) rejected softening the lock because of an asymmetric ratchet with **two** independent grounds:
   - **(A) The raise-back judgment is made by the dulled session.** The *up* call ("the hard turn is back") was made by the already-downgraded session, whose sensor the downgrade dulled. The gate framework removes this ground: it moves the **up** decision to the **user** (the router surfaces "this may need higher effort/model"; the user acts via `/effort` or `/model`), so the session no longer makes the call its dulling would corrupt. This ground is genuinely emptied.
   - **(B) Damage during the lowered window.** A lowered session runs at reduced power and does confidently-wrong work *before* anyone raises it back — "the export sensor is exactly the capability the downgrade dulled; a strong entrance only guards the valve, not the exit." The gate framework **does not empty this**: even with the user holding the raise-back decision, the lowered session still produces output during the window, and the user must first *notice* the output is wrong before raising. This is **not** structurally solved — it is accepted as **residual risk**, on the same terms as the effort-gate's residual (ADR-0011).

   So the reversal rests on ground (A) being emptied (the ratchet's *valve* no longer depends on a dulled sensor), while ground (B) is honestly demoted from "structural block" to "residual" rather than claimed solved. The net: lowering the main session is no longer *categorically* blocked, but its downside is real and survives as residual — which is exactly why the main-session floor stays at Sonnet, not Haiku (Haiku's lowered-window damage would be too large to accept even as residual).

## The new baseline model

The main session's baseline is **variable**, set by the user at session start, bounded:

- **Model ∈ {Opus, Sonnet}.** The main does not drop below Sonnet. Haiku is excluded from the main session: Haiku's residual risk (confidently-wrong output between a lower and the next raise) is too high for the long-lived accumulator, which has no executor/reviewer backstop of its own (its output goes straight to the user). (Haiku remains usable as a *spawned* subagent under the conditions in ADR-0011's spawn model-floor rule.)
- **Effort ∈ whatever the user set.** The router does not assume `high`; it treats the session-start effort as the baseline.

"Up" for the main session is now **two-dimensional** — effort (within the same model) or model (Sonnet→Opus). "Down" is symmetric in shape (either dimension) but, per the gate framework, lowering is not something the router proactively recommends; it's driven by a forward look and biased toward *not* lowering when unsure.

## Why the user decides the raise, not the router

The router cannot read its own runtime effort reliably (the status-line JSON omits effort — issue #36187; `max`/`ultracode`/model-default-hold are invisible to config reads — verified 2026-07-15), so it cannot distinguish "raise model" from "raise effort" at decision time. It therefore surfaces only the direction ("this may need higher power") and leaves the dimension choice to the user, who can see the current effort on the spinner. The router does not pretend to a capability (runtime introspection) the platform does not provide.

## Considered Options

- **Keep the Opus+high lock (the superseded ADR-0002).** Its window argument is void (point 1), its effort assumption was an overreach (point 2), and its ratchet defense is emptied by the gate framework (point 3). Three of its four legs removed; the fourth (meta-cognition is light at `high`) survives only as a *default*, not a *lock* — and the default belongs to the model, not the router.
- **Allow drop to Haiku on the main session.** Rejected: Haiku's residual risk is too high for the long-lived accumulator with no reviewer backstop. The main-session floor is Sonnet. (Haiku's use as a spawned subagent is governed separately, by the spawn model-floor rule, where reviewer separation applies.)
- **Let the router pick the raise dimension (model vs effort).** Rejected: it requires runtime introspection the platform doesn't expose, and distinguishing "needs higher ceiling (model)" from "needs more thoroughness (effort)" is a self-assessment of capability — the class ADR-0004 judges unreliable (confidently-wrong). The dimension choice goes to the user.

## Consequences

The main-session lock is gone; the baseline is a user-set variable within {Opus, Sonnet} × {user-set effort}. The router's gate becomes two-dimensional (raise model or effort), but its recommendation stays direction-only — the user picks the dimension, informed by the visible spinner. SKILL.md §2 (the Opus+high statements), §3 (the "only up is effort" and "locked Opus+high" clauses), and the Roster's framing are rewritten; CONTEXT.md's *Main session* and *Up-delegation* entries are updated. ADR-0002 is marked Superseded. The residual risk — a lowered session doing confidently-wrong work before the user raises it back — is the honest boundary of the design, carried on the same terms as the effort-gate's residual; it is not closed by more prompt.
