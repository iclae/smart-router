# Subagent templates

The types SKILL.md §2 and §3 select. **They do not work from here** — copy them into a directory Claude Code registers:

```bash
cp agents/effort-*.md agents/explore-haiku.md ~/.claude/agents/
```

Use `.claude/agents/` inside a project instead if you want them scoped to that project. Restart Claude Code afterwards if the target `agents/` directory did not already exist — the file watcher only covers directories present at session start.

The three `effort-*` bodies are **deliberately identical**; only the frontmatter differs. A subagent reads nothing but its own definition at spawn time, so the shared return discipline can't be factored out — a wording change must be applied to all three, or they drift silently.

What each template is for, when to pick which, and why any of this is a copy you install by hand: SKILL.md §3 and [ADR-0013](../docs/adr/0013-ship-subagent-templates-not-bundled-agents.md).
