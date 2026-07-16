# Subagent templates

The types SKILL.md §2 and §3 select. **They do not work from here** — copy them into a directory Claude Code registers:

```bash
cp agents/effort-*.md agents/explore-haiku.md ~/.claude/agents/
```

Use `.claude/agents/` inside a project instead if you want them scoped to that project. Restart Claude Code afterwards if the target `agents/` directory did not already exist — the file watcher only covers directories present at session start.

| Template | What it is |
|---|---|
| `effort-low.md` | Mechanical execution — decisions already made |
| `effort-medium.md` | Bounded deliberation — some judgment, none deep |
| `effort-high.md` | Heavy reasoning — the hard part is still ahead |
| `explore-haiku.md` | Read-only search on the cheapest model, returning its search terms |

Pass `model` at the spawn call for the `effort-*` three; name `explore-haiku` explicitly, as it cannot shadow the built-in `Explore`.

Why these are a copy rather than an install, why the tiers are semantic, why `explore-haiku` has to be named, and what about `effort:` is still unverified: [ADR-0013](../docs/adr/0013-ship-subagent-templates-not-bundled-agents.md).
