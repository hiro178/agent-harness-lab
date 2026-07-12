# Cumulative Deviation

Small per-step misalignments compound into total divergence.

## Symptom

No single step is obviously wrong, but after 10+ steps the result no longer matches the specification. Each step drifted a few degrees; the sum is a different destination. Reviewing any individual diff shows nothing alarming — the deviation only becomes visible when the current state is compared against the *original* spec, not against the previous step.

## Example

A refactoring task: step 3 renames a concept slightly ("session" → "context"), step 7 builds on the renamed concept with new semantics, step 12 adds behavior that only makes sense under the drifted semantics. Every diff was locally reasonable. The final module implements something the spec never asked for.

## Countermeasure

Periodic re-anchoring against the original specification:

- Insert **spec-compliance checkpoints** every N steps: compare the current state against the original requirements document, not against the last step.
- For orchestrated multi-agent work, give the orchestrator explicit checkpoint duties (verify against the plan before dispatching the next batch).
- Keep the spec in a file the agent re-reads at checkpoints — a spec that only lives in early conversation history is a spec that compaction will eventually erase (see [Goal Drift](goal-drift.md)).

## Distinct from

[Goal Drift](goal-drift.md) — there a single lossy summarization discards a constraint wholesale. Here each step is slightly off and the error accumulates gradually.

## Sources

- Fukushima (LayerX), "Agent Harness & AI Managed Services", 2026-04.
