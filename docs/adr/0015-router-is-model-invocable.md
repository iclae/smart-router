# smart-router is model-invocable: `disable-model-invocation` flipped off

> **Status** Accepted · **Relates to** [ADR-0011](0011-gate-direction-only-read-from-externals.md) (the same runtime blind spots this skill lives inside), [ADR-0013](0013-ship-subagent-templates-not-bundled-agents.md) (subagent load model, which bounds the residual below)
> **Live rule** SKILL.md frontmatter (absence of `disable-model-invocation`) and its `description`

The skill shipped with `disable-model-invocation: true`. That is the setting for side-effectful, timing-sensitive commands (`/deploy`, `/commit`) — you don't want the model deciding *when* they run. smart-router is the opposite: a pure reasoning discipline with no side effects, meant to apply at **every** task boundary. Setting it user-only created a structural tension ([#13](https://github.com/iclae/smart-router/issues/13) hole 3): SKILL.md says "run at every task boundary," but under `true` the model could not invoke the skill at all — the description never even entered its context — so the discipline was inert until the user remembered to type `/smart-router`. An easy link to forget, with no mechanism behind it.

We flip it off. With the field absent (platform default `false`), the description enters context and the model self-loads the discipline at the first task boundary that looks routing-relevant; from there the loaded body persists as a standing instruction covering every subsequent boundary (the platform's skill-content lifecycle). The bootstrap window shrinks from "until the user types the command" to "until the first substantive task" — and closes without any user action.

## Why not a SessionStart hook

The considered alternative was to keep `true` and add a `SessionStart` hook. It fails at a level below the fix:

- **A hook that injects a *reminder*** cannot make the model invoke a `disable-model-invocation: true` skill — that block is unconditional. So the reminder can only tell the *user* to type `/smart-router`. It automates the nag, not the activation.
- **A hook that injects the *full skill body* as `additionalContext`** does activate the discipline, but reintroduces exactly what ADR-0013 spent an ADR avoiding: machinery outside the skill file, a second install step, a config path that can drift. Worse, hook-injected text is ordinary transcript — it does not get the invoked-skill compaction protections (re-attach after summary, 25 K budget), so in a long session it can be summarized away with no signal, which is *worse* than the residual we already accepted.

Flipping the field needs no external machinery, keeps everything in the one SKILL.md, and is the only option that genuinely makes the discipline self-activating. It is also the grain of the platform: `disable-model-invocation` exists for side effects and timing control, neither of which this skill has.

## The subagent residual, accepted not guarded

Flipping to `false` opens one narrow door. A subagent is *not* auto-injected with this skill — the platform only preloads skills named in an agent's `skills:` field, and none of the shipped templates set it (ADR-0013); a non-fork subagent starts fresh and never inherits the parent's invoked skills or skill listing. But a subagent that holds the `Skill` tool *can* discover and invoke project/user skills mid-task, and under `false` smart-router becomes one of them (under `true` it was unreachable). So an `effort-*` or `general-purpose` subagent could, in principle, invoke the router inside itself — a delegatee playing router.

We accept this rather than guard it, on two grounds. It is improbable: a subagent works from a narrow "do X, return the artifact" brief and has no reason to reach for a routing meta-skill. And it is self-correcting: the skill's own description and §1 say it is the *main session's* discipline, so a subagent that did invoke it reads that it is the delegatee and stands down — the cost is a few wasted tokens, not the recursion or architecture break a real leak would cause. The description carries a one-clause reinforcement of that self-exclusion, which is the whole guard; a tool restriction would over-reach (subagents legitimately invoke other skills, e.g. TDD) and defends a scenario that both rarely fires and cleans up after itself — the kind of defense this project's simplicity discipline declines to write. (`explore-haiku` is already immune regardless: its tools are locked to Read/Grep/Glob, no `Skill`.)

## Considered Options

- **Keep `true`; add a SessionStart hook.** Rejected above — either it only nags the user, or it reintroduces ADR-0013's drift/second-install cost and loses compaction protection.
- **Flip to `false`; add a tool guard so subagents can't invoke it** (drop `Skill` from the templates' tools, or a deny rule). Rejected: over-restricts subagents that legitimately use other skills, to defend an improbable, self-correcting scenario.
- **Set `false` explicitly rather than removing the line.** Rejected as noise: `false` is the platform default, so stating it changes no behavior; this ADR is the record of the deliberate choice, and the diff (a deletion of `true`) already reads as "we turned this off."

## Consequences

The discipline now activates without user action, closing #13 hole 3's core. The description doubles as the trigger and the subagent self-exclusion notice, so it is load-bearing in a way it wasn't before — a future edit must preserve both. One residual stays open and is **not** closed here: in a long session, auto-compaction can drop the loaded skill when many other skills were invoked after it, with no signal (the compaction-loss residual noted during the #13 grill). That is a separate failure from the bootstrap gap this ADR fixes, and remains a known, accepted risk pending #13's verification-loop data.
