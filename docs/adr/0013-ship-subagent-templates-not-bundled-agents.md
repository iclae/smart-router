# Subagent definitions ship as templates the user installs, not as bundled agents

> **Status** Accepted · **Relates to** [ADR-0009](0009-effort-channel-is-subagent-file-not-spawn-param.md) (the channel this ADR has to deliver), [ADR-0012](0012-spawn-model-floor-tracks-the-reviewer-backstop.md) (the floor the types shipped here are spawned under)
> **Live rule** SKILL.md §3 (division-of-labour table), `agents/README.md` (install step)

SKILL.md §2 says a sized effort is realized by "selecting a subagent **type** pre-defined with it" (ADR-0009). That instruction is only executable if those types exist somewhere Claude Code registers them. They cannot ship with this skill.

**The platform fact** (verified 2026-07-17 against the official docs): subagent definitions are registered only from `.claude/agents/` (project), `~/.claude/agents/` (user), managed settings, the `--agents` CLI flag, or an installed **plugin**. Skills are a separate namespace: a `.md` file sitting in a skill's own directory is an inert resource, not a spawnable type. smart-router is distributed as a skill (`~/.claude/skills/smart-router/`), so it cannot carry its own subagent types — the one channel ADR-0009 identified as the only way to set a subagent's effort is a channel the skill has no access to.

So the four types this skill needs (`effort-low`, `effort-medium`, `effort-high`, and a cheap read-only explorer) ship as **templates** under `agents/`, and the user copies them into `~/.claude/agents/` once. The skill states the dependency; the platform install is the user's step.

## The cheap explorer has to be asked for by name

The grill's plan for the read-only explorer was to define a subagent also called `Explore`, letting it **shadow the built-in** so that exploration silently became cheap without the router having to choose. That mechanism does not work, and the replacement is to name `explore-haiku` explicitly (SKILL.md §3).

The evidence is one test plus one inference. A custom `Explore` was registered through the `--agents` CLI flag with a body that would have made the override unmistakable — reply with a fixed sentinel string — and the subagent that answered behaved like the built-in instead. Two explanations survive, and the test that would separate them (a lowercase control) was blocked by a session limit, so **which one is true is unresolved**. It does not matter here, because both close the same door:

- If the definition was rejected for its name — the docs require `name` to be lowercase letters and hyphens, and `Explore` is neither — then `name: Explore` is unshippable on its face.
- If the definition registered and lost anyway, then built-ins do not yield to same-named definitions at the `--agents` scope, which the docs rank at **priority 2**. The `.claude/agents/` files this ADR ships sit at **priority 3**. A scope that ranks lower than one already observed to lose cannot win.

So the shadowing mechanism is dead under either branch, and no amount of waiting on the control test revives it. What survives is the grill's *intent* — narrow read-only retrieval should run on the cheapest model, and should hand back its search terms so the main can review the query it can't see (ADR-0012). Only the mechanism changed.

The cost of the change is real and worth stating: shadowing would have made the cheap path the *default*, while explicit naming makes it a *choice the router has to remember*. The built-in `Explore` remains present and is the more obvious thing to reach for. This is a weaker design, chosen because the stronger one isn't available.

## Considered Options

- **Turn smart-router into a plugin** (add `.claude-plugin/plugin.json`; a skill folder with that manifest loads as a plugin and can bundle agents alongside the skill). This is the only option under which the skill is genuinely self-sufficient — install once, types present, no manual step, nothing to drift. **Deferred, not rejected on merit.** It changes what smart-router *is*: its distribution shape, its install instructions, and the name subagents appear under (plugin agents are scoped, e.g. `smart-router:effort-high`, which the division-of-labour table would have to name). That is a structural decision the map's grill never took, and taking it silently inside a landing ticket would be exactly the entropy this project's documentation discipline exists to prevent. It stays on the table as the obvious successor.
- **Ship nothing; have SKILL.md state the requirement** ("you need a subagent type declared at `effort: high`"). Rejected: it makes §2's instruction unfulfillable out of the box and pushes the whole design onto the reader, who has no way to know what the file body should say. The return-shape discipline in the template bodies is itself part of the decision (a subagent that narrates its process defeats the context economy that justified spawning it — ADR-0007); leaving it unwritten loses it.
- **Put the templates in `docs/agents/`.** Rejected: that directory holds guidance for the engineering skills that *read* this repo (`domain.md`, `issue-tracker.md`) — a different kind of artifact with a different audience. Reusing it would blur two module boundaries to save one directory.

## Consequences

The templates live in `agents/` at the repo root — deliberately the same layout a plugin uses, so the deferred plugin option is a manifest away rather than a reorganisation.

The honest cost: **the map's destination is only half met.** #10 asks that every SKILL.md instruction be supported by a landing artifact *that already exists*; under this decision the artifact exists but is not installed, so a fresh user who skips `agents/README.md` gets a skill whose §2 silently cannot execute — the router would size `(Haiku, low)`, try to select `effort-low`, and find nothing. That gap is the price of not taking the plugin decision inside this ticket, and it closes the day that decision is taken.

A second, quieter cost: the templates are now a **copy**, and copies drift. A user who installed them in July does not get a fix made in August. This is the same failure already visible in this repo — `~/.claude/skills/smart-router` is an independent clone stranded fourteen ADRs behind `main` — and the plugin option fixes both with one change.
