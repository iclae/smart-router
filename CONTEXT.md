# Smart Router

A user-invoked discipline that lets a main session delegate off-profile tasks to right-sized subagents, so heavy reasoning gets enough power and mechanical work doesn't burn it.

## Language

**Router**:
The skill itself — the standing discipline a main session self-applies to decide whether a task should run inline or be delegated to a subagent at a different profile.
_Avoid_: dispatcher, scheduler

**Profile**:
A (model, effort) pair, e.g. `Opus+medium`. The unit a task is sized against.
_Avoid_: tier, config, level

**Main session**:
The long-lived session the user works in. Fixed at `Opus+medium`. Plays router, orchestrator, and integrator — never the heavy laborer.
_Avoid_: parent, host

**Subagent**:
A cold-started agent the main session spawns for one off-profile task, returning a single compact artifact.
_Avoid_: child, worker, helper

**Up-delegation / Down-delegation**:
Spawning a subagent with more power (higher effort — Opus is already the top model) / less power (cheaper model, lower effort). Down is reliable; up self-assessment is weaker and leans on "round up when unsure".

**Roster**:
The single maintained list of currently-available models, ordered weakest→strongest, with each model's context-window size and caveats. A *list*, not a difficulty→model mapping — the only thing that goes stale when a vendor ships a new model.
_Avoid_: table, mapping, catalog

**Peak working set**:
The most a task must hold in context *at one instant* (a huge file, a broad sweep, many cross-referenced files) — as opposed to total volume. The quantity the context guard vetoes on.

**Veto layer** (context guard):
The rule applied *after* the difficulty judgment: never send a task whose peak working set may exceed a model's window to that model. Picks the cheapest model whose window covers the peak; when unsure, treat the working set as large.

**Load-bearing context**:
The facts a cold subagent cannot re-derive and so must be given. Everything else in the main session is noise and stays out.
_Avoid_: relevant context, background

**Brief**:
The four-part contract handed to a subagent: objective + return shape / load-bearing context (pointers over pastes) / boundaries / compact return format.

**Yield rule**:
When the current workflow is already owned by a skill that self-routes model/effort/spawn (linear-task-worker, devflow), the router stands down. One orchestrator at a time.

**Self-assessment**:
The router's difficulty check, fired only at task boundaries — each new user request, or when the main session carves off a clearly self-contained, clearly off-profile sub-block. Not per tool call.

**Verifiable criterion**:
A domain-agnostic check whose evaluation is *independent of* and cheaper than producing the work, yielding an observable pass/fail. The independence is what catches a confidently-wrong result — the check can't share the blind spot that produced it. General types: independent re-derivation, check-against-source, consistency/constraint, reproducible/executable, checklist coverage. Tests and typechecks are just the coding instances. Its absence is itself a routing signal — unverifiable + high-stakes routes to a human gate.
_Avoid_: success criteria, acceptance test (too coding/spec-bound)

**Executor/reviewer separation**:
The work and its check live in different contexts. Down-delegation gets this for free: the Opus main substantively reviews the weaker subagent's artifact against the criterion. Up-delegation cannot — main is the weaker party and can't review a result smarter than itself.
