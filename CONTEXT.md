# Smart Router

A user-invoked discipline that lets a main session delegate off-profile tasks to right-sized subagents, so heavy reasoning gets enough power and mechanical work doesn't burn it.

## Language

**Router**:
The skill itself — the standing discipline a main session self-applies to decide whether a task should run inline or be delegated to a subagent at a different profile.
_Avoid_: dispatcher, scheduler

**Profile**:
A (model, effort) pair, e.g. `Opus+high`. The unit a task is sized against.
_Avoid_: tier, config, level

**Main session**:
The long-lived session the user works in. Fixed at `Opus+high`. Plays router, orchestrator, and integrator — never the heavy laborer.
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

**Routing signal**:
The one-line, passive announcement the router emits at every task boundary stating the chosen outcome (e.g. `⟢ router: inline @ Opus+high`). A decision *log*, not a prompt — it asks nothing of the user. That is what separates it from the **gate**, an active escalation prompt ("consider /effort xhigh") that interrupts and asks the user to act. The `don't-nag` rule governs the gate, never the signal — so the signal fires even when the decision is "inline, unchanged", which is exactly the outcome that was previously invisible. For a high-consequence, low-reversibility inline task it also carries the verifiable criterion, because inline forfeits the free executor/reviewer separation.
_Avoid_: prompt, notification, log line

**Verifiable criterion**:
A domain-agnostic check whose evaluation is *independent of* and cheaper than producing the work, yielding an observable pass/fail. The independence is what catches a confidently-wrong result — the check can't share the blind spot that produced it. General types: independent re-derivation, check-against-source, consistency/constraint, reproducible/executable, checklist coverage. Tests and typechecks are just the coding instances. Its absence is itself a routing signal — unverifiable + high-stakes routes to a human gate.
_Avoid_: success criteria, acceptance test (too coding/spec-bound)

**Executor/reviewer separation**:
The work and its check live in different contexts. Down-delegation gets this for free: the Opus main substantively reviews the weaker subagent's artifact against the criterion. Up-delegation cannot — main is the weaker party and can't review a result smarter than itself.

**Context economy** (process-vs-result):
A spawn/inline driver co-equal with the model-power delta. When the main needs only a task's *result* and the process to reach it is verbose, doing it inline leaves that process squatting in the main context for the rest of a long session; a spawn keeps the process in the subagent and returns only the artifact. The mirror pull keeps work inline: when the process is worth *retaining* as basis for later work, a spawn would discard the trail the main needs. Distinct from the **executor/reviewer separation** a peer-spawn was rejected for (ADR-0005) — that was *capability* separation, which a same-profile spawn can't give; this is *context* separation, which it does. See ADR-0007.
_Avoid_: context hygiene (too one-directional — it cuts both ways)

**Re-read tax** (switching cost):
In Claude Code, changing the main session's model or effort makes the next message re-read the full accumulated history. Because the main is the long-context accumulator, the cost grows with how far the session has run — so the effort-gate is fired as early as heavy work is foreseeable, and a spawn (cold, no re-read) is preferred over a gate for any separable heavy task. See ADR-0006.
_Avoid_: switching overhead
