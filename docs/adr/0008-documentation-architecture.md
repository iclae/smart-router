# Documentation architecture: one home per fact

> **Status** Accepted · **Amended by** [ADR-0014](0014-five-document-types.md) · **Relates to** [ADR-0013](0013-ship-subagent-templates-not-bundled-agents.md) (the landing decision whose artifacts prompted ADR-0014's extension)

The skill's prose is itself a codebase whose modules are documents. Five document types each hold exactly one kind of fact, and no fact is duplicated across them — the last two were added by ADR-0014 once ADR-0013's landing produced artifacts none of the original three covered:

- **SKILL.md** — the executable procedure. Imperatives, thresholds, exact output strings, plus the *calibrating clause* of each judgment (which way to lean, where a threshold sits). Nothing else.
- **CONTEXT.md** — the glossary. One-line lookups: term → meaning + `_Avoid_`. Not a place to argue *why*.
- **`docs/adr/`** — the rationale. Full derivations, considered options, and reconciliations between decisions live here, one home each.
- **Template body** (`agents/effort-*.md`, `agents/explore-haiku.md`) — the executable action for a **subagent**, not the router. Charter and cut-rule in ADR-0014.
- **`agents/README.md`** — the install procedure for a **human** setting the templates up. Charter and cut-rule in ADR-0014.

## The cut-rule

A sentence stays in SKILL.md **iff removing it changes what the router *does*** at a task boundary. "Action" includes the calibrating clause — the one phrase answering *lean which way / how much* — because the router makes judgments, not mechanical steps, and a judgment without its calibration sends the reader to the ADRs at decision time. It excludes the full derivation and any cross-ADR reconciliation: removing those leaves the imperative plus its calibration intact, so they belong to the ADR alone.

This rests on a loading fact: when the skill is invoked only **SKILL.md's body** enters context; CONTEXT.md and the ADRs are linked and read on demand. So SKILL.md must stay self-sufficient *for action* — the one duplication that is load-bearing, not accidental.

## ADR status headers

Each ADR carries a blockquote header directly under its title: **Status**, **Amends / Amended-by** (the supersession chain), **Relates to** (load-bearing cross-references only, not passing mentions), and **Live rule** pointing to whichever document type holds the net current statement of the fact this ADR decided — SKILL.md for router action, a template body for subagent action, `agents/README.md` for install procedure, CONTEXT.md for a term's canonical definition — where one exists (generalized by ADR-0014; the original wording named SKILL.md exclusively, written when it was the only document type holding action). A reader landing on any single ADR can tell whether it is still in force and find its neighbours, without reconstructing the chain from prose.

## Considered Options

- **Self-contained SKILL.md (full rationale inline).** Rejected: it duplicates the ADRs, so a single decision change must be edited in three places — the locality failure this ADR exists to prevent. The cut-rule keeps only the action+calibration that single-file loading genuinely requires.
- **A synthesis doc stating the current spawn rule.** Rejected: it would duplicate SKILL.md §3 verbatim, re-creating the same locality violation one layer down. The live rule is stated once, in §3; the ADRs are its audit trail.
- **An ADR index / README.** Rejected for now: seven ADRs with descriptive filenames already list themselves, and the per-ADR status headers carry the supersession map locally. An index adds a module and a second maintenance point for a whole-chain overview nobody yet needs. Add it if the chain grows — one need is a hypothetical seam, not a real one.

## Consequences

Every fact has one home: a model change is a Roster edit, a decision change is one ADR, a procedure change is one SKILL.md step. A future architecture review must not re-suggest collapsing the load-bearing SKILL.md duplication, nor adding a synthesis doc, nor building an index — those are settled here. The standing cost is that writing now requires placing each fact deliberately; the cut-rule is the test that makes the placement mechanical. ADR-0014 extends the same principle to two more types without changing it — see that ADR for their charters and cut-rules.
