# Subagent templates

The types SKILL.md §2 and §3 select. **They do not work from here** — copy them into a directory Claude Code registers:

```bash
cp agents/effort-*.md agents/explore-haiku.md ~/.claude/agents/
```

Use `.claude/agents/` inside a project instead if you want them scoped to that project. Restart Claude Code afterwards if the target `agents/` directory did not already exist — the file watcher only covers directories present at session start.

Why a copy and not an install: skills and subagents are separate namespaces, and a skill cannot ship spawnable types. The full reasoning, and the plugin option that would remove this step, are in [ADR-0013](../docs/adr/0013-ship-subagent-templates-not-bundled-agents.md).

| Template | What it is |
|---|---|
| `effort-low.md` | Mechanical execution — decisions already made |
| `effort-medium.md` | Bounded deliberation — some judgment, none deep |
| `effort-high.md` | Heavy reasoning — the hard part is still ahead |
| `explore-haiku.md` | Read-only search on the cheapest model, returning its search terms |

The three `effort-*` templates declare `model: inherit`, so the model comes from the `model` argument at the spawn call and they stay pure effort containers — a vendor adding a model costs a Roster edit, not a file here (ADR-0001, ADR-0009). Their tiers are **semantic** (how much thinking the task needs), not a snapshot of which model/effort pairs are cost-efficient today. That snapshot lives in the Roster's note and goes stale; these do not.

`explore-haiku` pins `model: haiku` instead, and declares no effort — Haiku doesn't take the parameter. **Name it explicitly when you want it.** It does not override the built-in `Explore`: a same-named definition was tried and the built-in won, so the cheap explorer has to be asked for by name (SKILL.md §3).

Whether `effort:` has an observable runtime effect is **unverified** — the platform exposes no way to read a subagent's active effort, so nothing here has been confirmed to take effect. See the Roster note in SKILL.md and [#13](https://github.com/iclae/smart-router/issues/13).
