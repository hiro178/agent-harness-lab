# Context Pollution Cascade

One agent's subtly wrong output propagates into the next agent's input, and each stage amplifies it.

## Symptom

In a multi-agent pipeline, Agent A produces output that is 95% right. Agent B consumes it as ground truth and builds on the wrong 5%. Agent C consumes B's output and amplifies further. By the end of the pipeline the error is structural, and no single agent "made a mistake" from its own point of view — each faithfully processed what it inherited.

The root cause is missing ownership boundaries: agents don't know which parts of their context they **own** (may question, revise, verify) versus which parts they **inherit** (treat as immutable input). Without that contract, inherited errors are indistinguishable from requirements.

## Example

A research agent mislabels one API as deprecated. A planning agent, inheriting the research, routes the design around the "deprecated" API. An implementation agent then writes a worse workaround. Three correct-looking handoffs, one polluted pipeline.

## Countermeasure

Explicit context contracts between pipeline stages:

- Define per agent: **what it receives as immutable input, and what it is allowed to produce.** Ambiguity about ownership is where pollution enters.
- Insert an **independent evaluator between pipeline stages** to break propagation — a checker that validates stage N's output before stage N+1 consumes it (cheapest at the stage boundaries where claims become inputs).
- Mark low-confidence claims as claims (with source and confidence), not as facts, in inter-agent handoffs — downstream agents can then treat them as verify-before-use.

## Sources

- AL-Awady, 2026-05-23 — ownership-contract framing; convergent industry corroboration.
