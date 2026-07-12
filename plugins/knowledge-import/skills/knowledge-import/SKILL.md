---
name: knowledge-import
description: Use when a user asks to import, integrate, or incorporate a URL, document, or text block into the project — or to apply an imported best-practice checklist to audit real config/code. Runs external knowledge through triangulated verification before it's allowed to become or modify a project artifact (skill, agent, rule, or memory), and audits real artifacts against imported normative standards.
effort: high
---

# Knowledge Import — External Knowledge Structuring Pipeline

## Overview

A pipeline that turns external content (URLs, documents, text) into structured project artifacts — while gating the process on **source triangulation**, so a single unverified blog post can't silently mutate your rules or agent specs. Two outputs: new artifacts (Skill/Agent/Rule/Memory), or **integration** into existing artifacts, or — when the imported content is a normative standard rather than a design idea — an **audit-and-remediation** pass on your real config/code.

The core thesis: knowledge import is not "summarize and save." It's "verify, classify by what kind of change it should cause, and gate every mutation behind explicit checks."

## When to Use

- A URL, document, or text block needs to become an actionable project artifact
- External knowledge (blog posts, papers, style guides, runbooks) should extend an existing rule, skill, or agent spec — not just sit as a passive reference
- An external best-practice checklist or convention should be used to **audit and fix** a real artifact (config file, code, project structure)
- Batch import of multiple sources

## When NOT to Use

- Simple bookmark — save as a plain note/reference directly, skip the pipeline
- You already know the target artifact type and it's a trivial addition — just write it
- Content is purely project-specific context with no external claim to verify

## Phase 1: Triangulated Acquisition

Single-source fetch is insufficient for design-level knowledge import. This phase enforces source triangulation before extraction.

### Step 1a: Primary Acquisition

Fetch content from the user-supplied source. Fetcher selection follows [`references/source-routing.md`](references/source-routing.md) — a decision table mapping URL patterns to the right fetch strategy (rendered pages need a browser-capable fetch; long-form papers and PDFs need direct retrieval to avoid summarization bias; generic pages use the cheapest default). Keep routing logic in that file, not duplicated here — when you add a new domain pattern, edit only the table.

### Step 1b: Source Metadata Capture

Record before advancing to Phase 2 (all fields mandatory; empty fields block advance):

- `source_url`, `author`, `publication_date`
- `authority_tier`: `primary` | `secondary` | `tertiary` — assigned via the decision table below
- `pattern_matched`: which table row was matched (guards against tier gaming)
- `manual_override`: `null` by default; set only when the user explicitly overrides classification, with reason recorded verbatim
- `freshness_assessment`: `current` | `stale` | `timeless` (research papers are often timeless)

**authority_tier decision table**:

| Pattern | Tier | Examples |
|---------|------|----------|
| Peer-reviewed venue (arXiv, IEEE, ACM, Nature...) | `primary` | `arxiv.org/abs/*`, `ieeexplore.ieee.org` |
| Official spec/standard (RFC, W3C, ISO) | `primary` | `datatracker.ietf.org`, `www.w3.org/TR/*` |
| Vendor official documentation | `primary` | official product docs domains |
| Conference talk/keynote with recording | `secondary` | official conference-channel videos |
| Expert practitioner blog, named org affiliation | `secondary` | company engineering blogs |
| Personal blog / Medium / listicle | `tertiary` | personal publishing platforms |
| Unknown domain | `tertiary` (conservative default) | — |

**Anti-gaming constraint**: never mark a source `primary` without citing the `pattern_matched` row. `manual_override` to `primary` requires a user-provided reason recorded verbatim — the model cannot self-promote a source's tier.

If the source is `tertiary`, Step 1c requires fetching the primary source it cites (if any) or ≥1 independent source.

### Step 1c: Independent Source (Triangulation)

Obtain at least one independent corroborating source (user-supplied URL counts as source #1; total ≥ 2).

**Independent source definition**: different author AND different publishing organization, no direct citation chain (source B must not merely quote source A), ideally a different knowledge domain (academic + industry, research + practitioner).

- Two blog posts by the same author = 1 source, not 2.
- One paper plus five blog posts citing it = 1 independent claim (the paper is primary; the posts are downstream of it).
- An official source plus an independent practitioner blog reaching the same conclusion = 2 independent sources — the convergence strengthens confidence.

**Citation chain check**: fetch source B, grep its body for citation markers (`cite`, `reference`, `based on`, `according to`, author-name patterns) referencing source A's title/URL — a match means B is not independent of A.

**Organization equivalence check**: compare domain suffixes; same domain, or different domain with a matching author byline, counts as the same source republished, not a second source.

Cross-reference the key claims: do they agree, conflict, or add nuance? Record divergences in the Phase 2 extraction table with a `[Divergent]` tag; convergences need no special tag.

**Primary-tier exception**: if Step 1b recorded `authority_tier: primary`, Step 1c may be satisfied by the primary source alone (Gate 0 route B) — still attempt it, but don't block on a single primary source.

### Step 1d: Contradiction Scan vs. Existing Knowledge

Detect semantic conflicts between incoming knowledge and your existing project artifacts.

1. Select 2–5 key concepts from the Phase 2 draft extraction (or Phase 1a content if Phase 2 hasn't started).
2. Search each concept across your project's rule/memory/skill/CLAUDE.md locations — skip any that don't exist in this project.
3. **Absence handling**: a target that doesn't exist is recorded as "not scanned" — do not auto-tag it `[No reference]` (a target that's absent is a different fact than a target that's present-but-silent).
4. Read the relevant section of each hit and compare with the incoming claim.
5. Tag each cross-reference:

| Tag | Meaning | Downstream effect |
|-----|---------|-------------------|
| `[Aligned]` | Incoming reinforces existing | High integration value (evidence strengthening) |
| `[Complementary]` | Incoming extends existing without conflict | Medium integration value (extension candidate) |
| `[Conflict]` | Incoming contradicts existing | User must reconcile before Phase 3.5 |
| `[No reference]` | No relevant existing content | Standard Phase 3.5 handling |

This step checks **semantic consistency of content**; Phase 4 later checks **structural duplication of artifacts**. Both can fire on the same source and run independently.

**Iteration budget** (applies to Steps 1a–1d): hard cap 4 iterations per step, soft cap 8 total across Phase 1. Hitting the soft cap → report progress and ask continue-or-partial. Hitting a hard cap → stop that step, record partial state, surface to the user — never silently auto-advance.

## Phase 2: Knowledge Extraction

Extract structured knowledge into these categories:

1. **Key Concepts** — core ideas, principles, patterns
2. **Constraints** — rules, invariants, "always/never" statements
3. **Workflows** — step-by-step procedures, decision trees
4. **Code Examples** — reusable snippets, configuration patterns
5. **Role Definitions** — behavioral profiles, expertise domains
6. **Design Implications** — principles or failure patterns that should influence existing project design (routes to Phase 3.5)
7. **Normative Standards** — evaluation criteria, conventions, thresholds, or deny-lists that a **real artifact** (config, code, structure) should be measured against and brought into compliance with (routes to Phase 3.6, not 3.5)

**The 3.5 vs 3.6 discriminator**: a Design Implication is *descriptive* — you ADD it to a rule/agent/doc. A Normative Standard is *normative* — you AUDIT a real artifact against it and FIX the artifact, not the docs.

```markdown
| # | Category | Content | Confidence | Divergence (1c) | Alignment (1d) |
|---|----------|---------|------------|-------------------|--------------------|
| 1 | Constraint | "Always validate input at boundaries" | 0.90 | — | [Aligned] |
| 2 | Design Implication | "X pattern reveals a new failure mode" | 0.80 | — | [Complementary] |
| 3 | Normative Standard | "config MUST block dangerous commands" | 0.90 | — | [No reference] |
```

**Design Implication detection heuristic**: would removing this insight leave a gap in your project's design rationale? If yes, it's a Design Implication, not just a reference.

## Phase 3: Classification

Score candidate artifact types using the signal patterns and decision tree in [`references/classification-matrix.md`](references/classification-matrix.md). If the top-2 confidence scores are within 0.15 of each other, flag as **ambiguous** and ask the user rather than guessing. Check for multi-artifact potential — content can be rich enough to justify a primary plus a secondary artifact.

```markdown
## Classification Result
| Rank | Type | Confidence | Key Signals |
|------|------|------------|-------------|
| 1 | Skill | 0.82 | Step-by-step workflow, "How to" pattern |
| 2 | Rule | 0.45 | Some constraints present |

**Decision**: Primary = Skill (gap = 0.37, no ambiguity)
```

## Phase 3.5: Integration Analysis

**Purpose**: before generating new artifacts, check whether extracted insights should be **integrated into existing artifacts** instead of stored as standalone references. Design principles and failure patterns often belong inside an existing rule or skill, not as an isolated new file.

For each item categorized as Design Implication or Constraint:

1. **Map to existing artifacts** — search your project's rules/agents/skills/CLAUDE.md/memory for related content.
2. **Assess fit** — does this extend, refine, or add a failure mode to something that already exists?
3. **Score integration value** on three metrics:

| Metric | 1.0 pt | 0.5 pt | 0 pt |
|--------|--------|--------|------|
| lines_delta (estimated addition size) | ≥5 lines | 2–4 lines | ≤1 line |
| failure_modes (new cases captured) | ≥1 | — | 0 |
| actionability | complete rule/example | concept clear, needs a concrete form | background only |

`integration_score = sum of the three`. ≥2.0 = HIGH, 1.0–1.5 = MEDIUM, <1.0 = LOW.

```text
For each extracted insight:
├── integration_score ≥ 2.0 → Integration action (modify existing artifact)
├── integration_score 1.0–1.5 → Integration + Reference Memory (both)
├── No mapping found, but actionable → New artifact (Phase 5)
└── No mapping, not directly actionable → Reference Memory only
```

**Integration execution**: read the target artifact, draft the diff, **present it to the user with rationale before applying**, apply only after approval, record it in the output report. Integration must respect the target's existing structure and conventions — extend it, don't restructure it.

## Phase 3.6: Standard Compliance Audit & Remediation

**Purpose**: when the imported content is a *normative standard* (evaluation criteria, thresholds, deny-lists, checklists), don't stop at storing or integrating it — **audit your project's real artifacts against it and remediate the divergence in place**. This is the "fix the actual config/code" path, distinct from 3.5's "extend the design docs" path.

**Trigger**: Phase 2 produced a Normative Standard item AND a real artifact exists to audit it against. No normative standard, or nothing to audit → skip; the standard routes through 3.5 / Reference Memory instead.

| | Phase 3.5 Integration | Phase 3.6 Remediation |
|--|--|--|
| Knowledge type | Descriptive design insight | Normative evaluation criteria |
| Target of change | Docs / rules / agents | Config / code / structure |
| Direction | **Add** to existing docs | **Conform** the real artifact to the standard |
| Verification | Structure preserved, addition reads naturally | Divergence closed, sections preserved, diff churn bounded |

### Step A — Audit

Reduce each standard to a **machine-checkable** probe and record the current state: size checks (`wc`), presence/absence (`grep`), structured config (`jq`). **Formality Gradient rule**: only formally-verifiable criteria become a probe. Judgment-based criteria (tone, intent, "is this still meaningful") stay as prose findings for the user to assess — never force a judgment call into a pass/fail probe; that enforces the letter and misses the intent.

```markdown
| # | Standard | Probe | Current | Verdict | Remediation needed |
|---|----------|-------|---------|---------|--------------------|
| 1 | deny-list MUST block dangerous ops | grep the deny config | absent | violation | YES |
| 2 | size ≤ N | wc -c | under limit | compliant | no |
```

### Step B — Remediation Plan

Order violations by priority × risk × reversibility. Classify each: **LOW** (reversible, local, no shared-state effect) · **MED** (reversible but impactful) · **HIGH** (irreversible or affects shared state). For each, record the target, the exact change, the risk class, and the verification probe.

### Step C — Safe Apply

1. **Dry-run first** — print the complete list of every target (file + key/line + before→after) before touching anything.
2. **Backup for reversibility** — a copy or a clean git checkpoint before any edit.
3. **Structure-based editing only** — edit via structured tools (`json.load`/`dump`, `jq`, or single-site text edits). Never bulk-rewrite by line number: an earlier edit shifting offsets silently corrupts a later line-numbered write.
4. **Verify after apply** — re-run each Step A probe, confirm the artifact still parses, confirm unrelated sections are untouched, confirm diff churn is proportional (no accidental reformat).
5. **Scope-strict shipping** — stage only the remediated artifact(s) explicitly. Never sweep unrelated working-tree diffs into the commit.

**User gate**: HIGH risk requires explicit approval before applying. LOW/MED risk proceeds after presenting the dry-run. Record what was remediated in the final report.

## Phase 4: Deduplication [gate: fail = block]

This is control flow, not a suggestion.

1. **Name search** — check existing artifact names (skills/agents/rules) and memory index entries for exact or near matches.
2. **Semantic search** — check for conceptual overlap if you have a semantic search tool available.
3. **Description comparison** — does an existing artifact already cover this?

| Result | Action |
|--------|--------|
| Exact match | BLOCK — report the existing artifact, suggest updating it instead |
| Partial overlap | WARN — present the overlap, ask: merge or create new? |
| No overlap | PROCEED |

## Gate 0: Triangulation (Pre-Generation)

Sits between Phase 4 (Deduplication) and Phase 5 (Generation) — a standalone control point enforcing Phase 1's triangulation requirement before anything gets generated or mutated.

**Passes when any route holds**:

| Route | Condition |
|-------|-----------|
| (A) Multi-source | ≥2 independent sources (user-supplied URL counts as #1) |
| (B) Primary-single | Single source AND `authority_tier = primary`, matched via the Step 1b table (not via `manual_override`) |
| (C) Reference Memory downgrade | User explicitly approves downgrading — bypasses Phase 3.5 integration |

**Common prerequisites**: Step 1b metadata complete (all fields, `pattern_matched` cited) · Step 1d contradiction scan run, any `[Conflict]` resolved.

**Failure protocol** (A and B both fail) — present all three options, no auto-selection:

| Option | Action | Required user signal |
|--------|--------|----------------------|
| Retry triangulation | Fetch one more independent source | explicit "try another source" |
| Correct tier | Re-run the Step 1b table against the URL; only promote if a `primary` pattern actually matches | explicit request to re-check |
| Downgrade to Reference Memory | Skip Phase 3.5, store as memory with an audit tag | explicit "reference memory only" |

**Non-interactive environments** (CI/batch, no interactive user prompt available): on Gate 0 failure, log the reason, auto-downgrade to Route C semantics (skip 3.5, continue to 4/5 as memory-only, tag `triangulation_status: downgraded`), and continue rather than hang waiting for input that will never come.

**Anti-gaming constraints**: `manual_override` may only tighten tiering (tertiary → secondary), never bypass Gate 0(B) by self-promoting to primary without a matching table row. The model must not auto-select the downgrade option — that decision belongs to the user. Downgraded imports skip Phase 3.5 entirely: un-triangulated knowledge never mutates existing rules/skills/agents, only ever lands as tagged reference memory.

## Phase 5: Generation

Generate using [`references/artifact-templates.md`](references/artifact-templates.md). Two modes: **Generation** creates new artifacts, **Integration** modifies existing ones — Phase 3.5 already decided which applies to each item.

**For Integration**: read the target, draft the specific diff (new section/table row/failure mode), preserve existing formatting and conventions, present the diff before applying, apply only after approval.

**For new artifacts**: match whatever authoring conventions your project already uses for that artifact type (skill/agent/rule format, language, structure). This skill does not prescribe your project's conventions — it enforces that content reaches generation only after triangulation and deduplication passed.

## Phase 6: Validation Gates [all fail = block]

Control flow enforcement, not optional checks.

- **Gate 1 — Frontmatter/metadata validation**: required fields present, format parses without errors, naming conventions followed.
- **Gate 2 — Structure validation**: correct placement in your project's artifact layout; any file references the artifact makes actually resolve.
- **Gate 3 — Deduplication recheck**: Phase 4 result is PROCEED or user-approved WARN; no new artifacts were created by something else during generation.
- **Gate 4 — Convention compliance**: matches your project's established language and formatting conventions; no obvious band-aid patterns in code examples (silent type-erasure, disabled lint rules, empty catch blocks).

If any gate fails: block, report which gate and why, do not proceed.

## Common Mistakes

- **Generating without deduplication** — always run Phase 4 first; duplicate artifacts waste context budget.
- **Skipping validation gates** — gates are mandatory control flow.
- **Forcing a single artifact type** — rich content often produces a primary plus a valid secondary artifact; don't discard the secondary.
- **Over-generating** — not every paragraph needs its own artifact. Merge related concepts.
- **Storing when integrating is better** — if an insight directly extends an existing rule or skill, integrate it there instead of spawning a standalone reference.
- **Auto-applying integrations** — integration diffs must be presented to the user before applying. Never auto-modify existing artifacts.
- **Scope leakage in remediation** — staging everything in the working tree after a remediation sweeps in unrelated diffs. Stage only the remediated artifact(s) explicitly.
- **Editing a real artifact with no dry-run and no backup** — Phase 3.6 Step C mandates both; editing config/code blind has no rollback path.
- **Line-based bulk rewrites** — rewriting a file by line offsets corrupts it once an earlier edit shifts line numbers. Use structured tools or single-site edits.
- **Over-mechanizing a judgment standard** — forcing tone/intent/"is this still meaningful" into a pass/fail probe enforces the letter and misses the point. Only formally-verifiable standards get a structural probe.
