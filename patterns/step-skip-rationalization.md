# Step-Skip Rationalization

The agent generates a plausible justification for skipping a verification or planning step.

## Symptom

"I'm confident this change is correct, so running the full suite isn't necessary." "This is too simple to test." "The type checker would catch any issue here." The rationalization is always articulate and often locally reasonable — that is exactly what makes it dangerous. The steps most likely to be skipped are the ones that exist to catch the agent's own blind spots.

## Example

An agent fixing a "trivial" one-line bug skips the reproduction step because the cause "is obvious from reading the code". The obvious cause is wrong; the real bug is in a caller. The fix ships, the symptom persists, and a second session burns an hour rediscovering what a two-minute reproduction would have shown.

## Countermeasure

Pre-enumerate the excuses and their counters — don't argue in the moment:

- Build **anti-rationalization tables** into skill and workflow design: a two-column list of common skip-justifications ("too simple to test", "I already verified mentally", "time pressure") paired with the mandated counter-action.
- Treat rules as constitution, not suggestion: a verification step that can be waived by the worker's own judgment is advisory, and advisory steps get rationalized away under exactly the conditions where they matter most.
- Where possible, move the step out of the agent's discretion entirely — make it harness control flow (see [Completion Misidentification](completion-misidentification.md)).

## Sources

- Addy Osmani, agent-skills — anti-rationalization tables in skill design.
- Block Engineering, "Constitution, not Suggestion" — convergent finding on advisory vs. mandated steps.
