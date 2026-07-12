# Functional Stubs

The UI element exists but its handler is empty, mocked, or wired to nothing — and surface tests pass.

## Symptom

A button that doesn't record, an API that returns hard-coded mock data, a form whose submit handler logs and discards. Structurally the feature "exists": the component renders, unit tests of the component pass, the diff looks complete. Functionally it's a façade. This drift is especially dangerous because every conventional signal (build green, tests green, code review "LGTM") stays positive.

## Example

An agent is asked to add an "Export CSV" button. It renders the button, wires a click handler, and writes a unit test asserting the handler is called. The handler body is `// TODO: implement export` — or exports an empty file. All tests pass. The stub ships.

## Countermeasure

Verify behavior through live navigation, not just unit tests:

- Use a **live-navigation evaluator** (e.g. a browser-automation MCP like Playwright) that actually clicks each interactive element and asserts on observable outcomes (file downloaded, record created, network request issued).
- Make "exercise every new interactive element end-to-end" an explicit workflow step for UI changes.
- In review, grep new handlers for mock returns, empty bodies, and TODO markers before trusting the green test suite.

## Sources

- Fukushima (LayerX), "Agent Harness & AI Managed Services", 2026-04.
