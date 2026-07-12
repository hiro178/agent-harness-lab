---
name: drift-patterns
description: Use when designing an agent harness or autonomous workflow, reviewing why an autonomous agent silently went wrong, or auditing a multi-agent pipeline for reliability. Catalogs 10 drift patterns (silent agent failure modes) with structural countermeasures — completion misidentification, self-grading, goal drift after compaction, verifier theater, phantom confabulation, and more.
---

# Drift Patterns: Silent Agent Failure Modes

Agents rarely fail loudly. This skill catalogs the 10 recurring ways autonomous agents silently go wrong, and the countermeasure for each. Apply it in two directions: **design-time** (build the countermeasure into the workflow before the failure happens) and **diagnosis-time** (match observed misbehavior to a pattern, then apply its fix).

## Diagnosis: match the symptom first

| Observed symptom | Likely pattern |
|------------------|----------------|
| "Done!" but tests never ran / criteria unmet | 1. Completion Misidentification |
| "I reviewed my own changes, looks good" | 2. Quality Self-Overconfidence |
| Each step fine, final result off-spec | 3. Cumulative Deviation |
| A "don't do X" constraint vanished mid-session (esp. after compaction) | 4. Goal Drift |
| Feature "exists" but handler is empty/mocked; tests green | 5. Functional Stubs |
| Articulate excuse for skipping verification ("too simple to test") | 6. Step-Skip Rationalization |
| Multi-agent pipeline amplifies an upstream error | 7. Context Pollution Cascade |
| Agent presents options the workflow never defined | 8. Emergent Menu Drift |
| Tests pass because the tests were weakened | 9. Verifier Theater |
| Agent reports an attack or a dialogue that never happened | 10. Phantom Confabulation |

## The patterns and countermeasures

### 1. Completion Misidentification

Agent declares "done" prematurely; confidence comes from having written code, not observed it work.
**Fix**: external completion gates — test execution as workflow control flow (step N = run tests, failure = block), done-condition checked by a separate process, never self-certified.

### 2. Quality Self-Overconfidence

The same context that produced the work reviews it, inheriting every blind spot.
**Fix**: Generator/Evaluator separation — review in a separate agent call with fresh context and a different objective (security-focused, correctness-focused), never as a continuation of the working session.

### 3. Cumulative Deviation

Per-step misalignments compound; each diff looks fine, the sum diverges from spec.
**Fix**: periodic spec-compliance checkpoints comparing current state against the *original* spec (kept in a file that gets re-read), not against the previous step.

### 4. Goal Drift

Load-bearing constraints ("must NOT touch legacy/") silently drop, especially after compaction — summaries discard constraints first.
**Fix**: re-inject constraints after each compaction from a durable file; for long tasks prefer orchestrating separate context windows with focused briefs; mirror critical constraints into machine-enforced permissions.

### 5. Functional Stubs

UI element renders, handler is a stub or mock; unit tests pass, feature is a façade.
**Fix**: live-navigation evaluation — actually exercise each new interactive element end-to-end (browser automation) and assert on observable outcomes; grep new handlers for mock returns and TODO bodies.

### 6. Step-Skip Rationalization

Plausible justification for skipping verification/planning ("I'm confident", "too simple to test").
**Fix**: anti-rationalization tables — pre-enumerate the excuses with mandated counter-actions; move critical steps out of agent discretion into harness control flow.

### 7. Context Pollution Cascade

Agent A's subtly wrong output becomes Agent B's ground truth; C amplifies. Root cause: no ownership contract on shared context.
**Fix**: explicit context contracts per agent (immutable inputs vs. producible outputs); independent evaluator between pipeline stages; mark claims as claims (source + confidence) in handoffs.

### 8. Emergent Menu Drift

Agent widens a fixed decision menu (2 options → 3+), routing users into untested paths. Reads as UX improvement; is a stage-contract violation.
**Fix**: hard-code completion templates per stage; treat menu-shape divergence as stage failure. The workflow author owns the option set, not the agent.

### 9. Verifier Theater

A separate verifier exists but "passes" work by rewriting/weakening failing tests or asserting on mocks — the oracle is corrupted.
**Fix**: strip test-edit permission from the verifier role; require independent test *execution* with explicit PASS/FAIL evidence; in CI/loops, run tests from a clean checkout the maker cannot mutate.

### 10. Phantom Confabulation

Under long loops the agent confabulates a non-existent reality: "quotes" injected instructions that exist only in its own earlier outputs (phantom injection), or fabricates an interlocutor and awaits their approval (phantom dialogue, with correction immunity). Mechanism: persistent outputs outlive ephemeral reasoning — later turns read earlier speculation as documented evidence.
**Fix**: forensics over narrative — grep the raw transcript for the claimed payload counted by message role (payload only in `assistant` ≈ self-generated); but check out-of-band channels (inter-agent messages, MCP, hooks) before concluding, since they bypass the visible log. For phantom dialogue, content-based correction fails: do a procedural reset (dump state to files, cross-check external records like git log, restart from verified ground truth). Bound loop length with session resets.

## Design-time checklist

When building an autonomous workflow, verify each is answered structurally (in the harness), not by trusting the agent:

- [ ] Who certifies completion? (must not be the worker — #1)
- [ ] Who reviews quality? (fresh context, different objective — #2)
- [ ] When is the original spec re-read? (checkpoints — #3, #4)
- [ ] Are new UI/API surfaces exercised live? (#5)
- [ ] Can verification steps be waived by the worker's judgment? (they must not — #6)
- [ ] Do pipeline stages have context ownership contracts? (#7)
- [ ] Are stage decision menus hard-coded? (#8)
- [ ] Can the verifier modify the tests? (it must not — #9)
- [ ] Is loop length bounded, with transcript forensics available? (#10)

Design implication: assume 10–20% of autonomous sessions will drift. Compensate with parallel execution and structural gates, not by relaxing constraints.

## Full catalog

Extended write-ups with examples and sources: <https://github.com/hiro178/agent-harness-lab> (`patterns/` directory).
