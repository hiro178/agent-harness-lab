# Completion Misidentification

The agent declares "done" before the work is actually done.

## Symptom

"All done — the feature is implemented and working!" while tests were never executed, half the acceptance criteria are unmet, or the change was never exercised at runtime. The agent's confidence is generated from having *written* code, not from having *observed* it work.

## Example

An agent implements a new API endpoint, reports completion, and moves on. The handler compiles, but nobody ever sent a request to it. An integration test would have revealed a missing route registration — but no gate forced that test to run before "done" could be declared.

## Countermeasure

External completion gates. The agent must not be the party that certifies its own completion:

- Make test execution a **workflow step**, not a prompt suggestion ("step N = run tests; failure = block") — the agent cannot skip what the harness enforces.
- Require a completion checklist whose items are verified by command output (exit codes, test reports), not by the agent's narrative.
- For autonomous loops, have a separate process (or a separate small-model check) evaluate the done-condition each turn instead of trusting the worker's self-report.

The design rule: **don't ask the agent whether it is done; make the workflow decide.**

## Sources

- Fukushima (LayerX), "Agent Harness & AI Managed Services", 2026-04 — drift-pattern taxonomy this catalog builds on.
