# Goal Drift

The original objective loses fidelity across many turns — especially after context compaction.

## Symptom

Edge-case requirements and "don't do X" constraints silently disappear from the agent's working understanding. The headline goal survives ("build the importer") but the load-bearing qualifiers ("…without touching the legacy schema", "…idempotent on retry") are gone. The trigger is often compaction: every summarization is lossy, and constraints are what summaries drop first — they read like details.

## Example

A long session starts with "migrate the config format; the old format must keep working for one release". After two compactions, the compatibility constraint is no longer in context. The agent, working correctly against its *current* understanding, deletes the old-format parser. Nothing looked wrong from the inside.

## Countermeasure

Treat constraints as state to be re-injected, not history to be remembered:

- **Re-inject load-bearing constraints after each compaction** — keep them in a file or system-level context that survives summarization, and have the workflow re-read it.
- For long adversarial or highly structured tasks, prefer **orchestrating separate context windows** with focused, isolated goals over one long-running context. A fresh window with a precise brief cannot drift on constraints it was just given.
- Write constraints in negative-space form where possible ("must NOT modify legacy/") and mirror the critical ones into machine-enforced permissions so that drift cannot express itself as action.

## Distinct from

[Cumulative Deviation](cumulative-deviation.md) — there per-step misalignment compounds gradually. Here one lossy summarization discards a constraint wholesale.

## Sources

- Thariq & Sid Bidasaria (Anthropic), "A harness for every task: dynamic workflows", 2026-06-02 — compaction loss and separate-context orchestration.
