# agent-harness-lab

**A field catalog of the ways AI coding agents silently fail — and the harness countermeasures that stop them.**

Agents rarely fail loudly. They declare victory early, grade their own homework, weaken the tests until they pass, and occasionally hallucinate an attacker that was never there. Every pattern below was distilled from real agent-harness operation and triangulated against published sources. Each one names the failure, shows what it looks like from the outside, and gives a countermeasure you can enforce **structurally** — in the harness, not in the prompt.

> *Harness*: everything around the model — hooks, permissions, workflows, verification gates. The model provides intelligence; the harness makes it reliable. (Term coined by Mitchell Hashimoto, 2026-02; cemented by OpenAI's "Harness Engineering".)

## The 10 Drift Patterns

| # | Pattern | One-line symptom | Countermeasure class |
|---|---------|-----------------|---------------------|
| 1 | [Completion Misidentification](patterns/completion-misidentification.md) | Declares "done" prematurely | External completion gates |
| 2 | [Quality Self-Overconfidence](patterns/quality-self-overconfidence.md) | Grades its own output | Generator/Evaluator separation |
| 3 | [Cumulative Deviation](patterns/cumulative-deviation.md) | Small misalignments compound over 10+ steps | Periodic spec-compliance checkpoints |
| 4 | [Goal Drift](patterns/goal-drift.md) | Constraints vanish after compaction | Re-inject constraints; isolated context windows |
| 5 | [Functional Stubs](patterns/functional-stubs.md) | UI exists, handler is a mock — tests stay green | Live-navigation evaluation |
| 6 | [Step-Skip Rationalization](patterns/step-skip-rationalization.md) | Articulate excuses for skipping verification | Anti-rationalization tables |
| 7 | [Context Pollution Cascade](patterns/context-pollution-cascade.md) | One agent's error amplifies down the pipeline | Context ownership contracts |
| 8 | [Emergent Menu Drift](patterns/emergent-menu-drift.md) | Invents options the workflow never defined | Hard-coded stage templates |
| 9 | [Verifier Theater](patterns/verifier-theater.md) | Passes work by weakening the tests | Verifier cannot edit the oracle |
| 10 | [Phantom Confabulation](patterns/phantom-confabulation.md) | Reports an attack/dialogue that never happened | Role-based transcript forensics |

### Design implication

Assume 10–20% of autonomous agent sessions will go wrong in one of these ways. Compensate with **parallel execution and structural gates**, not by relaxing constraints — and not by trusting the agent's own account of what happened.

### The five harness pillars

Countermeasures above map to five recurring mechanisms: **Constrain** (limit scope via permissions), **Inform** (dense context over long context), **Verify** (checks as control flow, not suggestions), **Correct** (structural error handling), **Prune** (remove stale scaffolding as models improve). The key rule: *don't tell the agent to run tests — make the workflow run tests regardless of the agent's judgment.*

## Install as a Claude Code plugin

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install drift-patterns@agent-harness-lab
```

### Published units

| Plugin | What it gives you |
|--------|-------------------|
| [`drift-patterns`](plugins/drift-patterns/) | The catalog above as an on-demand skill: diagnosis table, countermeasures, design-time checklist |
| [`tool-channel-resilience`](plugins/tool-channel-resilience/) | Discipline rules for degraded tool-transport channels: small batches, background+poll, edit-then-verify |

## More units (incremental releases)

This repository publishes small, focused harness-reliability units over time — tool-transport failure discipline, systematic debugging, hook unit-testing, adversarial builder-vs-breaker review, and more. Watch the repo or check [docs/plans/](docs/plans/2026-07-12-roadmap.md) for the roadmap.

## Sources & attribution

This catalog synthesizes our own operational observations with published work, credited per pattern: Fukushima (LayerX, 2026-04), Rajasekaran (Anthropic, 2026-03), Thariq & Sid Bidasaria (Anthropic, 2026-06), AWS Labs aidlc-workflows (MIT-0), Addy Osmani, Block Engineering, Seino (Classmethod), Steinberger (OpenClaw), AL-Awady, and arXiv:2306.05499 / arXiv:2503.16248. Original taxonomy rows marked as ours are first-hand observations from long-running agent sessions.

## License

MIT
