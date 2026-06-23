# Spawn boundary, peak-working-set veto, and orchestrator yield

Three rules bound when the router acts:

1. **Spawn only on dual condition** — the needed profile differs from `Opus+medium` enough to matter *and* the task is a self-contained sub-problem whose result returns as a compact artifact. A hard task that is the main through-line is not spawned (it would make the weaker main a relay for a result it can't evaluate). A floor applies: if the task is small enough that main just does it, the fixed brief+cold-start cost outweighs the saving — don't spawn.

2. **Peak-working-set veto** — applied *after* the difficulty judgment. Never route a task whose peak simultaneous working set may exceed a model's window to that model. Choose the cheapest model whose window covers the peak; when the working set is unknown, treat it as large. This vetoes on *peak*, not total volume — high-volume/low-working-set tasks (rename across many files, run a test suite) stay safely on cheap models.

3. **Yield to orchestrators** — when the current workflow is already owned by a skill that self-routes model/effort/spawn (linear-task-worker, devflow), the router stands down. One orchestrator at a time.

## Consequences

Discipline does not cross the spawn boundary for free: a cold subagent does not inherit the skill or methodology the main session is running (e.g. TDD), so any methodology the subagent must honor is carried as load-bearing context in the brief. Self-assessment fires only at task boundaries (new user request, or a carved-off off-profile sub-block), never per tool call.
