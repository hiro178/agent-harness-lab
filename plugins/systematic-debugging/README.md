# systematic-debugging plugin

Root-cause-first debugging as an enforceable discipline — extended for agent workflows.

## What's different from the well-known version

The four-phase core (investigate → pattern-match → hypothesize → implement) follows the systematic-debugging discipline from [obra/superpowers](https://github.com/obra/superpowers) (MIT), credited in the skill. This plugin adds four **agent-oriented extensions** that autonomous and semi-autonomous workflows need:

| Extension | What it adds |
|-----------|--------------|
| **Phase 3.5 — Root cause classification** | A 10-category taxonomy (`LOGIC_ERROR`, `CONFIG_GAP`, `SPEC_CONFLICT`, …) that makes debugging output machine-comparable across sessions and agents |
| **Phase 3.6 — STOP_FOR_HUMAN protocol** | Pre-defined escalation conditions (spec conflicts, credential boundaries, public-API breakage) with a fixed handoff format — the agent knows *when it must not decide alone* |
| **Phase 3.7 — Structured output** | A fixed report format (`ROOT_CAUSE` / `CATEGORY` / `FIX_PLAN` / `VERIFICATION` / `NEXT_ACTION` / `CONFIDENCE`) so parent agents and orchestrators can consume debugging results programmatically |
| **Phase 5 — Question reframing** | After 3 failed fix attempts, stop debugging the answer and debug the question: "how do I make X faster?" → "should X happen at all?" Structural elimination beats individual optimization |

## Install

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install systematic-debugging@agent-harness-lab
```

## When it activates

On demand when you hit a bug, test failure, or unexpected behavior — before any fix is proposed. It is designed to be most valuable exactly when you least want to follow it: under time pressure, with an "obvious" quick fix in hand.
