# Judgment lives in the model, facts live in a roster list

> **Status** Accepted

The difficulty→model decision is delegated to the main model at runtime, not encoded as a maintained difficulty→model table. The only maintained artifact is a **roster**: a flat list of current models ordered weakest→strongest, each with its context-window size and caveats.

## Considered Options

- **Maintained difficulty→model mapping table.** Rejected: it bakes a snapshot of model intelligence into the skill, and goes silently wrong the moment a vendor upgrades or re-tiers models — exactly the failure we're guarding against.
- **Fully model-judged, no roster.** Rejected: a model's knowledge cuts off at a date, so it cannot know which models exist now or their windows. Letting it pick a model id from memory reintroduces the staleness, hidden inside the model's head where it's harder to spot.

## Consequences

The split: the model judges difficulty live (auto-adapts as models get smarter); the roster supplies the only thing the model can't know — the current lineup and each model's window. A vendor change is a one-line roster edit, judgment logic untouched. The roster carries a "last checked" date because the facts genuinely move (e.g. Sonnet 4.6 gained a 1M window months after a Jan-2026 cutoff).
