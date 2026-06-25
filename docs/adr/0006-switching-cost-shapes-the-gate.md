# Switching cost shapes the effort-gate

Changing the main session's model or effort mid-stream makes Claude Code re-read the full accumulated history on the next message (cache invalidated, all input tokens reprocessed). The main session is the long-context accumulator (ADR-0002), so this **re-read tax** scales with how far the session has run — a late switch on a near-1M context is expensive in both latency and usage credits.

The router induces exactly one kind of main-session switch: the effort **gate** (the main is locked to Opus, so the model never changes; `/effort` is the only knob, and it's user-level). So the re-read tax is, specifically, a cost of the gate. Three rules follow:

1. **Gate early.** Raise the gate at the first task boundary where heavy work is foreseeable, not after the session has piled up context. The same effort bump is near-free at the start and expensive at the end.
2. **Prefer a spawn over a gate for separable heavy work.** A subagent starts cold from a compact brief — it never touches the main history, so it pays zero re-read tax (only brief + cold-start, on a tiny context). A gate re-reads the entire main history. So when the heavy work is separable, a spawn now dominates the gate on cost too — on top of the executor/reviewer separation it already wins (ADR-0004). The gate is the fallback for *non-separable* through-line heavy work.
3. **A late, non-separable gate must name its cost.** When a gate does fire deep into a session, its message says so ("this re-reads the full history") so the user decides with eyes open.

## Consequences

The re-read tax is also a third independent argument — alongside the big-window and cheap-meta-cognition arguments in ADR-0002 — for locking the main profile rather than switching it per task: the very property that makes the main valuable (it accumulates everything) is what makes switching it expensive. It reinforces, but does not by itself establish, the lock.
