# tool-channel-resilience plugin

*[English](README.md) | [日本語](README_ja.md)*

Discipline rules for surviving a degraded tool-transport channel — the layer between an AI agent and its tool execution.

## The problem

Sometimes the transport itself misbehaves: tool results come back empty or truncated, a trivial `echo ok` probe returns nothing, a whole batch of tool calls gets cancelled at once, or the response halts mid-tool-call. Agents (and users) instinctively respond by retrying harder or adding prompt instructions like "be careful with tool calls" — neither works, because **this is an infrastructure condition, not a behavior problem**.

## The thesis: bound the blast radius

Instructions cannot make a flaky transport reliable. What they *can* do is control how much damage one failure causes:

- **Small batches** — one malformed call cancels its whole batch, so fewer calls per batch = smaller blast radius.
- **Background + poll** — detach heavy commands from the fragile synchronous channel entirely.
- **Edit-then-verify** — catch silent file corruption after each edit, while it is still one `git checkout` away from recovery.
- **Checkpoint durable state** — when the channel is truly down, commit what exists and tell the user, instead of retrying blindly.

The skill ships a detection checklist (how to tell a degraded channel from ordinary errors), eight discipline rules with rationale, and the canonical background+poll pattern with macOS/BSD-portable commands.

## Install

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install tool-channel-resilience@agent-harness-lab
```

## When it activates

On demand, when tool calls start failing in the patterns above — or preventively, when running long autonomous sessions where transport hiccups are costly.
