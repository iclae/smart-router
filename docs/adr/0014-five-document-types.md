# Documentation architecture extended: two more document types, generalized Live rule

> **Status** Accepted · **Amends** [ADR-0008](0008-documentation-architecture.md) · **Relates to** [ADR-0013](0013-ship-subagent-templates-not-bundled-agents.md) (the landing decision this amendment reconciles)

ADR-0008 named three document types, each holding exactly one kind of fact: SKILL.md (router action), CONTEXT.md (glossary), `docs/adr/` (rationale). ADR-0013's landing shipped artifacts that fit none of the three — subagent-facing template bodies (`agents/effort-*.md`, `agents/explore-haiku.md`) and a human-facing install procedure (`agents/README.md`) — and said so in its own Consequences: "the architecture is not broken, but it is now under-described." That amendment was deliberately deferred to its own ticket rather than taken inside a landing ticket. This is that ticket.

## Two new charters

- **Template body** (`agents/effort-*.md`, `agents/explore-haiku.md`) — the executable action for a **subagent**, not the router. A sentence stays iff removing it changes what the subagent does when it executes. This is a *stricter* version of SKILL.md's loading fact: a subagent at spawn time reads only its own definition file — not SKILL.md, not CONTEXT.md, not the ADRs — so the return-shape discipline must be fully self-contained in the body, with nothing left to inherit. That is why the three `effort-*` bodies are deliberately identical (`agents/README.md` already states the constraint; this charter is why it's ADR-0008's business rather than an installation footnote). The charter excludes why the router picks this type over another (SKILL.md §3's job) and why the template is shaped the way it is (ADR-0013's job) — a template body carries no rationale, only instruction.
- **`agents/README.md`** — the install procedure for a **human** setting the templates up. A sentence stays iff removing it changes what a person installing the templates does. Excludes rationale (ADR-0013) and router action (SKILL.md §3) — the two places it was twice caught drifting toward during #12's code review (cited in #14's issue body).

## Live rule, generalized

ADR-0008's original Live rule convention pointed exclusively at "the SKILL.md section that holds the net current statement" — written when SKILL.md was the only document type that held action. ADR-0013 needed a Live rule for `agents/README.md`'s install procedure and had nowhere ADR-0008-compliant to point it, so it pointed there anyway and flagged the mismatch rather than silently violating the convention.

The fix generalizes the rule instead of special-casing the new type: a **Live rule points to whichever of the five document types holds the net current statement of the fact this ADR decided** — SKILL.md for router action, a template body for subagent action, `agents/README.md` for install procedure, CONTEXT.md for a term's canonical definition where the decision is a naming choice, or no Live rule where no single document holds a "current form" (ADR-0008 and this ADR are both examples: they describe the architecture itself, which no one document executes).

## Considered Options

- **Give `agents/README.md` its own header field ("Install rule") instead of generalizing Live rule.** Rejected: it keeps "Live rule always means SKILL.md" intact for readers already used to it, but buys that by giving one document type a bespoke pointer mechanism — exactly the per-type special case ADR-0008's "one home per fact" exists to prevent. A single generalized rule — "current statement's home, whichever type that is" — covers all five without adding a field.
- **Leave the architecture at three types; treat `agents/` as outside its scope.** Rejected: ADR-0013 already shipped artifacts governed by no cut-rule, and the absence was not hypothetical — `agents/README.md` drifted toward ADR territory twice during #12's review, caught only because a human reviewer was reading closely both times.

## Consequences

ADR-0008 now lists five document types and states the generalized Live rule; its own header carries `Amended by ADR-0014`. ADR-0013's header drops the "declares this architecture under-described" annotation on its `Relates to [ADR-0008]` line and the "not a SKILL.md section — see Consequences" caveat on its `Live rule` line — both resolved here, not caveats any more. No other ADR changes: the generalization is additive, and every existing `Live rule` pointing at SKILL.md still reads correctly under the new wording (SKILL.md is still "the document type holding router action" in every case that named it before).
