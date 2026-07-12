# Quality Self-Overconfidence

The agent grades its own output — and the grade is always generous.

## Symptom

A self-grading loop: the same context window that produced the work also reviews it, and the review inherits every assumption and blind spot of the original. "I reviewed my changes and they look good" carries close to zero information.

## Example

An agent writes an authentication middleware, then "reviews" it in the same session and finds nothing wrong. A reviewer with fresh context immediately spots that the token-expiry branch was never handled — the original context had already rationalized it away as out of scope.

## Countermeasure

Generator/Evaluator separation. The evaluator must have no investment in the generator's approach:

- Run the review as a **separate agent call with fresh context** — not a continuation of the working session.
- Give the evaluator a different objective function (e.g. a security-focused or correctness-focused prompt) rather than "check my work".
- Anthropic's `security-guidance` plugin is a worked example: a separate Claude call, fresh context, security-focused prompt — the reviewer never saw the reasoning that produced the code, so it cannot inherit its rationalizations.

## Distinct from

[Verifier Theater](verifier-theater.md) — there a *separate* verifier exists but corrupts the test oracle. Here the problem is that no separate verifier exists at all.

## Sources

- Rajasekaran (Anthropic), "Harness Design for Long-Running Apps", 2026-03 — Generator/Evaluator separation.
- Anthropic `security-guidance` plugin — implementation example of the fresh-context evaluator.
- Fukushima (LayerX), "Agent Harness & AI Managed Services", 2026-04.
