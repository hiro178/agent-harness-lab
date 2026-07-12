---
name: systematic-debugging
description: Use when hitting a bug, test failure, or unexpected behavior — BEFORE proposing any fix. Enforces root-cause-first debugging with a 10-category cause taxonomy, a STOP_FOR_HUMAN escalation protocol, structured output for parent agents, and question reframing after 3 failed attempts. Especially use under time pressure, when a "quick fix" seems obvious, or when previous fixes didn't work.
---

# Systematic Debugging

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle: ALWAYS find root cause before attempting fixes. Symptom fixes are failure.**

> Attribution: the four-phase core (Phases 1–4) follows the systematic-debugging discipline from [obra/superpowers](https://github.com/obra/superpowers) (MIT). Phases 3.5–3.7 and Phase 5 are this project's agent-oriented extensions — built for autonomous and semi-autonomous agent workflows where the debugger must classify, escalate, and report structurally.

## The Iron Law

```text
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Any technical issue: test failures, production bugs, unexpected behavior, performance problems, build failures, integration issues.

**Use this ESPECIALLY when:** under time pressure (emergencies make guessing tempting) · "just one quick fix" seems obvious · you've already tried multiple fixes · the previous fix didn't work.

**Don't skip when** the issue seems simple (simple bugs have root causes too) or you're in a hurry (rushing guarantees rework).

## The Phases

Complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

1. **Read error messages carefully** — full stack traces, line numbers, error codes. Don't skip past warnings.
2. **Reproduce consistently** — exact steps, reliable trigger. Not reproducible → gather more data, don't guess.
3. **Check recent changes** — git diff, recent commits, new dependencies, config changes.
4. **Gather evidence in multi-component systems** — log what enters/exits each component; verify config propagation; run once to see WHERE it breaks.
5. **Trace data flow** — where does the bad value originate? Keep tracing up to the source. Fix at source, not at symptom.

### Phase 2: Pattern Analysis

Find working examples of similar code in the same codebase · read the reference implementation COMPLETELY · list every difference, however small · understand what other components this depends on.

If the codebase has no comparable working example, fall back to the language's or framework's standard idiom for the pattern and diff against that — the comparison target changes, the discipline doesn't.

### Phase 3: Hypothesis and Testing

Form a **single** hypothesis ("I think X is the root cause because Y") · test with the SMALLEST possible change, one variable at a time · didn't work? Form a NEW hypothesis — do NOT stack more fixes on top · when you don't know, say "I don't understand X" instead of pretending.

### Phase 3.5: Root Cause Classification

Classify the confirmed root cause with a standard category — this makes debugging output machine-comparable across sessions and agents:

| CATEGORY | Description | Typical fix |
|----------|-------------|-------------|
| `MISSING_DEPENDENCY` | Package/module not installed or not imported | Add dependency, fix import |
| `RUNTIME_MISMATCH` | Version conflict between runtime expectations | Pin version, update compatibility |
| `MODULE_FORMAT` | ESM/CJS/TS module resolution issue | Fix tsconfig, add type:module |
| `NATIVE_ABI` | Native binary incompatibility | Rebuild, use pure-JS alternative |
| `CONFIG_GAP` | Missing or incorrect configuration | Add/fix config entry |
| `LOGIC_ERROR` | Incorrect algorithm or business logic | Fix logic, add edge-case handling |
| `TASK_ORDERING_PROBLEM` | Steps executed in wrong dependency order | Reorder dependencies |
| `TASK_DECOMPOSITION_PROBLEM` | Task too large or mis-scoped | Split task, redefine scope |
| `SPEC_CONFLICT` | Implementation contradicts specification | Resolve with spec owner |
| `EXTERNAL_DEPENDENCY` | Third-party API/service behavior changed | Update integration, add adapter |

Classification note: performance limits rooted in **workload design** (the job does more work than the goal allows, even though every operation is individually correct) classify as `TASK_DECOMPOSITION_PROBLEM`, not `LOGIC_ERROR` — the logic is right, the decomposition of what gets processed is wrong.

### Phase 3.6: STOP_FOR_HUMAN Decision

Escalate to the human **immediately** when ANY of these hold:

- Root cause is `SPEC_CONFLICT` — a human must decide which side is correct.
- Root cause is `EXTERNAL_DEPENDENCY` with no workaround — needs account/API access.
- The fix requires credentials, permissions, production config, or schema changes you don't have or shouldn't touch.
- 3+ fix attempts failed AND the reframed question (Phase 5) still has no clear path.
- The fix would break a public API contract or change user-visible behavior.

```text
STOP_FOR_HUMAN
CATEGORY: {category}
ROOT_CAUSE: {description}
REASON: {why human judgment is needed}
OPTIONS: {paths for the human to choose from}
```

### Phase 3.7: Structured Output

Report debugging results (to humans or parent agents) in this format:

```text
ROOT_CAUSE: {one-line description of the actual cause}
CATEGORY: {one of the categories above}
FIX_PLAN: {specific steps, with file paths}
VERIFICATION: {exact command to verify the fix}
NEXT_ACTION: {FIX | STOP_FOR_HUMAN | REFRAME}
CONFIDENCE: {HIGH | MEDIUM | LOW}
NOTES: {caveats, related issues, prevention suggestions}
```

### Phase 4: Implementation

1. **Create a failing test case** — MUST exist before fixing.
2. **Implement a single fix** — ONE change at a time; no "while I'm here" improvements.
3. **Verify** — test passes? No other tests broken?
4. **If the fix doesn't work** — STOP. Under 3 attempts: return to Phase 1. At 3+: go to Phase 5.

### Phase 5: Question Reframing (after 3+ failed approaches)

When Phases 1–4 have failed 3+ times, the problem is likely not the solution — it's the question.

"Failed" means **the goal is still not met** — this includes attempts that worked locally (each hypothesis confirmed, each fix effective) but moved the needle far less than the goal requires. Three correct fixes that close 9% of a required 80% gap are three failures.

1. **Challenge the premise**
   - "How do I make X faster?" → "Should X happen at all?"
   - "How do I fix this error?" → "Why does this code path exist?"
   - "How do I handle this edge case?" → "Can the design eliminate this edge case?"
2. **Individual optimization vs structural elimination**
   - Individual: making N operations each faster saves `N × Δt`.
   - Structural: eliminating `(1-p) × N` operations entirely saves `(1-p) × N × t`.
   - Structural elimination almost always wins.
3. **Reframing checklist**
   - [ ] Can the operation be eliminated rather than optimized?
   - [ ] Can the caller avoid needing this result?
   - [ ] Can the architecture make this problem impossible?
   - [ ] Is a different abstraction level the right place to solve this?
4. **Record the reframe** — original question, reframed question, why the original framing was limiting. This insight is often the most valuable output of the session.

Example from practice: 14 attempts to optimize dequantization operations all failed. Reframing "how to dequantize faster" into "skip dequantization entirely via a sparse V-cache" yielded a +22.8% throughput improvement.

## Red Flags — STOP and return to Phase 1

"Quick fix for now, investigate later" · "Just try changing X and see" · "Add multiple changes, run tests" · "It's probably X, let me fix that" · "I don't fully understand but this might work" · "One more fix attempt" (when already tried 2+).

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. The process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | The first fix sets the pattern. Do it right from the start. |
| "Multiple fixes at once saves time" | You can't isolate what worked — and you cause new bugs. |
| "One more fix attempt" (after 2+) | 3+ failures = the question is wrong. Go to Phase 5. |

## Quick Reference

| Phase | Key activities | Success criteria |
|-------|---------------|------------------|
| 1. Root cause | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| 2. Pattern | Find working examples, compare | Differences identified |
| 3. Hypothesis | Single theory, minimal test | Confirmed or new hypothesis |
| 3.5–3.7 | Classify, escalate if needed, report structurally | CATEGORY + structured output |
| 4. Implementation | Failing test, one fix, verify | Bug resolved, tests pass |
| 5. Reframing | Challenge premise after 3 failures | New question or structural elimination |

## Supporting techniques

Root-cause tracing, defense-in-depth validation, and condition-based waiting (for flaky tests) are covered in depth by [obra/superpowers](https://github.com/obra/superpowers) — this skill intentionally does not duplicate them.
