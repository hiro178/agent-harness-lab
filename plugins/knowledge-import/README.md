# knowledge-import plugin

*[English](README.md) | [日本語](README_ja.md)*

A pipeline for turning external content into project artifacts — gated on source triangulation, not "summarize and save."

## The problem with naive knowledge import

Ask an agent to "import this article as a rule" and it will happily read one blog post and mutate your CLAUDE.md. One unverified source now has write access to your project's design rationale. This plugin adds the missing gate: **content only reaches generation after it clears source triangulation, deduplication, and structural validation** — all enforced as control flow, not as prompt suggestions the model can rationalize past.

## What makes this different from "read a link and summarize it"

- **Authority tiering + triangulation** (Phase 1): sources are tiered (primary/secondary/tertiary) against an explicit decision table, and anti-gaming constraints stop the model from self-promoting a source's tier. Single-source tertiary imports are blocked from mutating anything until a second independent source is found — or the user explicitly accepts a downgrade.
- **Integration vs. new artifact vs. audit-and-fix** (Phases 3.5/3.6): the pipeline distinguishes three outcomes that are usually conflated — extend an existing rule with a new failure mode, generate a standalone new artifact, or (when the imported content is a normative standard) **audit your real config/code against it and remediate the divergence**, with dry-run, backup, and risk-classed approval gates.
- **Deduplication as a blocking gate** (Phase 4): exact-match imports are blocked outright, not silently duplicated.
- **Validation gates before anything ships** (Phase 6): frontmatter, structure, dedup-recheck, convention compliance — all fail-closed.

## Install

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install knowledge-import@agent-harness-lab
```

## When it activates

When you ask to import, integrate, or incorporate external content into the project, or to audit a real artifact against an imported best-practice checklist.

Full pipeline detail — routing table, classification matrix, remediation procedure — lives in [`skills/knowledge-import/SKILL.md`](skills/knowledge-import/SKILL.md) and its `references/`.
