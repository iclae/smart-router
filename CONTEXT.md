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
The long-lived session the user works in. Its baseline is a **variable** the user sets at session start — model ∈ {Opus, Sonnet} (never Haiku), effort whatever the user chose — not a router-fixed `Opus+high`. The router adapts to the baseline it finds; it does not assume a fixed value. Plays router, orchestrator, and integrator — never the heavy laborer.
_Avoid_: parent, host

**Subagent**:
A cold-started agent the main session spawns for one off-profile task, returning a single compact artifact.
_Avoid_: child, worker, helper

**Up-delegation / Down-delegation**:
Spawning a subagent with more power / less power. **Down** = cheaper model or lower effort (reliable; main reviews the artifact — ADR-0004 layer 2). **Up** is now two-dimensional on the main session: raise effort (same model) or raise model (Sonnet→Opus). The main cannot self-escalate either (`/effort`, `/model` are user-level), so up is surfaced as a gate and the dimension choice is left to the user.

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
When the current workflow is already owned by an orchestrator skill — one that self-routes model/effort/spawn, whatever it's called — the router stands down. One orchestrator at a time.

**Self-assessment**:
The router's difficulty check, fired only at task boundaries — each new user request, or when the main session carves off a clearly self-contained, clearly off-profile sub-block. Not per tool call.

**Routing signal**:
The one-line, passive announcement the router emits at every task boundary stating the chosen outcome (e.g. `⟢ router: inline @ <baseline>`) — a decision *log*, not a prompt, as opposed to the **Gate**.
_Avoid_: prompt, notification, log line

**Gate**:
The router's one active outcome: a *prompt* interrupting the user to ask that the main session's power be raised — as opposed to the passive **Routing signal**. Recommends a direction, never a target level or a dimension, because the router cannot read its own runtime effort. Its shape and bounds are SKILL.md §3, its derivation ADR-0011.
_Avoid_: warning, alert, nag

**Task externals**:
The properties the gate reads to decide "this needs more power" — verifiability (the strongest), consequence and reversibility, peak working set, shape — all visible before the work starts, and so readable when felt strain isn't (ADR-0004). Applied only once the task is known to be a through-line that can't be spawned off; separable work is a spawn question, not a gate one.
_Avoid_: signals, symptoms, difficulty cues

**Probe**:
The bounded look the router may take while judging a gate: read-only tools answering the **Task externals** checklist, stopped the moment they're answered. Not the start of the work being judged.
_Avoid_: investigation, spike, recon

**Verifiable criterion**:
A domain-agnostic check whose evaluation is *independent of* and cheaper than producing the work, yielding an observable pass/fail — the independence is what catches a confidently-wrong result, which shares the blind spot that produced it. The type menu and the gate-the-unverifiable rule live in SKILL.md §6.
_Avoid_: success criteria, acceptance test (too coding/spec-bound)

**Spawn model floor**:
The weakest model a task may be spawned to: Haiku when a verifiable criterion is nameable *and* the main will check the artifact against it, Sonnet when either fails (ADR-0012).
_Avoid_: minimum model, model floor (ambiguous — the main session has one too)

**Executor/reviewer separation**:
The work and its check live in different contexts. Down-delegation gets this for free: the Opus main substantively reviews the weaker subagent's artifact against the criterion. Up-delegation cannot — main is the weaker party and can't review a result smarter than itself.

**Context economy** (process-vs-result):
A spawn/inline driver co-equal with the model-power delta, cutting both ways: spawn when the main needs only the *result* and the process to reach it is verbose (it would otherwise squat in the main context); keep inline when the process is worth *retaining* as basis for later work.
_Avoid_: context hygiene (too one-directional — it cuts both ways)

**Falsifiable spawn prediction**:
The clause a **context-economy** spawn's routing signal carries, naming what the main will use going forward (`[X]`); falsified if the main later reaches back for the spawned-away process. Layer 1 of #13's verification loop — power-delta spawns don't carry it. Emission in SKILL.md §3, derivation in ADR-0016.
_Avoid_: prediction, guess, estimate

**Re-read tax** (switching cost):
In Claude Code, changing the main session's effort makes the next message re-read the full accumulated history; because the main is the long-context accumulator, the cost grows with how far the session has run. A fact of the **gate** (the one main-session switch the router induces), not a routing condition — the full two-sided derivation lives in ADR-0002 (now Superseded by ADR-0010; the re-read-tax *fact* survives, only its former role as a routing condition was retired).
_Avoid_: switching overhead
