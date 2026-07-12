# Emergent Menu Drift

The agent invents a wider decision menu than the workflow specified.

## Symptom

A workflow stage specifies a fixed 2-option completion menu ("Request Changes" / "Continue to Next Stage"). The agent, trying to be helpful, presents 3 or 5 options — adding "Skip this stage", "Do both", "Let me refine first". It reads as a UX improvement. It is actually a **stage-contract violation**: the user's decision surface has silently widened beyond what the workflow author designed and validated, and the extra options route into untested paths.

## Example

A staged spec-review workflow expects binary approval per stage. The agent offers "approve with minor reservations" as a third option; the user takes it; downstream stages have no defined semantics for a half-approval and silently treat it as full approval. The reservation is lost.

## Countermeasure

The workflow author — not the agent — owns the option set:

- **Hard-code completion message templates per stage.** The agent fills in content, never the option structure.
- Treat menu-shape divergence as a **stage failure**, not an emergent improvement: if the presented options differ from the spec, the stage did not complete correctly.
- Write "NO EMERGENT BEHAVIOR" expectations explicitly into workflow definitions — agents interpret silence as permission to improvise.

## Sources

- AWS Labs, aidlc-workflows `core-workflow.md` ("NO EMERGENT BEHAVIOR"), MIT-0 — primary source of the pattern.
- devtoaaron (dev.to) — convergent practitioner reading.
