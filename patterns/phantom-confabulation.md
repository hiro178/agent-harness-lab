# Phantom Confabulation

Under a long loop, the agent confabulates a non-existent reality and reports it as fact.

> Naming note: "Phantom Confabulation" is our label for this pattern. The underlying mechanisms are corroborated by the sources below; the unified framing is ours.

## Symptom

Two observed variants:

- **Phantom Injection** — the agent reports having *been* prompt-injected, "quoting" hidden malicious instructions. Forensics show the quoted payload exists only in the agent's **own earlier assistant-role outputs**: it speculated about an attack, then later read its own speculation as evidence of one.
- **Phantom Dialogue** — the agent fabricates an interlocutor and a conversation that never happened: it answers prompts nobody sent, and compaction progressively reframes solitary work as collaboration ("awaiting your approval on the previous step"). Attempts to correct it get reinterpreted as *continuation of the phantom dialogue* — **correction immunity**.

## Why it persists

Persistent text outlives ephemeral reasoning. The agent's speculative outputs remain in the transcript; the reasoning that produced them ("this is just a hypothesis") does not. A later turn — or a post-compaction summary — reads the earlier speculation as documented fact. Each loop iteration re-confirms the confabulated reality from its own residue.

## Countermeasure

Don't trust the narrative; audit the record:

- **Grep the raw transcript for the claimed payload, counted by message role.** If the "injected instructions" appear only in `assistant` messages (≈0 hits in `user` or tool results), the attack is self-generated.
- **Known blind spot:** role-grep cannot see out-of-band delivery (inter-agent messaging queues, MCP responses, hook-injected context bypass the visible conversation log). Log-absence is *not* confabulation-confirmed — check every delivery channel before concluding.
- For phantom dialogue, content-based correction fails (correction immunity). Use a **procedural reset**: dump durable state to files, cross-check against external records (git log, issue tracker), and restart the session from verified ground truth.
- Treat pre-malfunction outputs as the baseline, and **bound loop length** with periodic session resets — confabulation risk grows with unbroken loop duration.

## Distinct from

[Goal Drift](goal-drift.md) — that is constraint *loss* through lossy compaction. Here the agent *adds* a false reality. Also distinct from real prompt-injection attacks: in this pattern there is no attacker and no interlocutor — the agent's own perception is the false positive. The forensic protocol above is exactly how you tell the two apart.

## Sources

- arXiv:2306.05499 — prompt-injection mechanics (what real injection looks like, against which forensics compare).
- arXiv:2503.16248 — persistence asymmetry: outputs persist, reasoning does not.
- anthropics/claude-code issue #40166 — out-of-band message delivery invisible in logs (the forensic blind spot).
- anthropics/claude-code issue #64698 — a phantom turn carrying a real payload (why role-grep alone is insufficient).
