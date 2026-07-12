---
name: tool-channel-resilience
description: Use when tool calls start failing strangely — empty or truncated tool results, trivial probe commands returning nothing, whole tool batches cancelled at once, or responses halting mid-tool-call. Provides discipline rules that bound the blast radius of a degraded tool-transport channel and recover gracefully. Instructions cannot fix transport failures; they can only contain them.
---

# Tool-Channel Resilience

The tool-output channel — the transport between the agent and tool execution — can suffer intermittent failures: empty or truncated tool results, probe commands returning nothing, whole tool batches cancelled. This is a **transport/infrastructure** condition. Instructions CANNOT fix it. They can only **bound the blast radius** and **recover gracefully**.

## Detect

Treat the channel as degraded when any of these recur:

- A trivial probe (`echo ok`) returns empty / no output.
- Shell or file-read results truncate or come back blank repeatedly.
- A multi-tool batch fails wholesale with no per-tool diagnostic.
- A response halts mid-tool-call, or tool syntax comes back malformed, immediately after a long text preamble.

## Disciplines

| Discipline | Rule | Why |
|------------|------|-----|
| **Small batches** | Under instability, ≤3 tool calls per turn — never 10+. | One malformed call can cancel the **entire** batch; concurrency multiplies blast radius. |
| **Isolate user questions** | Never bundle an interactive user-question tool in a multi-tool batch. | If a sibling call is malformed, the whole batch — the question included — is cancelled and the user never sees it. |
| **Text–tool separation** | Tool-emitting turns carry ≤1 short sentence. Put insights, explanations, and preambles in a tool-free turn or *after* tool results — never as a long preamble immediately before a tool call. | Long prose co-emitted with a tool call raises both max-token mid-call truncation *and* malformed tool-block serialization (the response "stops midway"). The failure tracks text length adjacent to the call. |
| **Heavy commands → background + poll** | Run long/heavy commands (full test suite, build, E2E) in the background, then poll for a sentinel; never foreground-block. | Foreground blocking on a flaky channel times out or returns empty; backgrounding detaches execution from the fragile synchronous response. |
| **Edit-then-verify** | After each file edit, verify the change with `grep` or AST inspection (not by re-reading the whole file). On corruption, restore with `git checkout HEAD -- <file>`. | Bounds damage to one edit; catches silent truncation/duplication before it compounds. |
| **No bulk line-based writes** | Never bulk-insert by line number (e.g. a script rewriting a file by offsets). Use targeted string-match edits, one site at a time. | Line-number writes corrupt files when earlier edits shift offsets — an entire class of silent duplication bugs. |
| **Portable commands (macOS/BSD)** | No `timeout` (GNU-only), no `cat -A`. Prefer POSIX-portable forms. `TZ=… date` is fine. | On macOS these return `RC=127` or get cancelled, poisoning the whole batch. |
| **Resume re-derivation** | On resuming any interrupted or scheduled loop, re-derive actual state (`git log`, `git status`, `grep`) before acting — do not trust a carried-over "in-progress" narrative. | A resumed turn may drag stale mid-task state; verify ground truth first. |

## Canonical background + poll pattern

Run the heavy command in the background, detached from the synchronous channel, and have it write a sentinel on exit:

```bash
# run in background — writes a sentinel on exit
<heavy-cmd> > ./tmp/run.log 2>&1; echo "RC=$?" >> ./tmp/run.log
```

Then, in subsequent turns, poll for the sentinel instead of using a wall-clock kill (`timeout` is unavailable on macOS):

```bash
until grep -q "RC=" ./tmp/run.log; do sleep 15; done; tail -n 40 ./tmp/run.log
```

## What this does NOT do

This bounds blast radius and enables recovery — it does **not** prevent transport-layer flakiness. If `echo ok` itself returns empty repeatedly, the channel is down: drop concurrency to 1, **checkpoint durable state** (commit / write the deliverable to disk), and surface the condition to the user rather than retrying blindly. Reading is not verification; a silent-empty result is not a passing result.
