# Classification Matrix — Knowledge Import

## Decision tree

```text
Input Content
├── Contains passive constraints ("always X", "never Y", "must/shall")?
│   ├── YES → Primary: Rule
│   │   └── Also has project-specific context? → Secondary: Memory
│   └── NO ↓
├── Contains step-by-step procedure or workflow?
│   ├── YES → Primary: Skill
│   │   └── Also has passive constraints? → Secondary: Rule
│   └── NO ↓
├── Contains role definition, behavioral profile, or expertise domain?
│   ├── YES → Primary: Agent
│   │   └── Also has procedural workflow? → Secondary: Skill
│   └── NO ↓
├── Contains project-specific context, config, or settings?
│   ├── YES → Primary: Memory
│   │   └── Also has constraints? → Secondary: Rule
│   └── NO ↓
└── Ambiguous or mixed → Flag for user confirmation
```

## Signal pattern table

| Signal pattern | Primary artifact | Confidence | Examples |
|---|---|---|---|
| "Always/Never/Must/Shall" + constraint | **Rule** | 0.85 | Style guides, coding standards, security policies |
| Numbered steps, "How to X", workflow | **Skill** | 0.80 | Blog tutorials, process docs, runbooks |
| "As a [role]", behavioral traits, expertise | **Agent** | 0.75 | Job descriptions, persona specs, team charters |
| Project names, env configs, team decisions | **Memory** | 0.70 | Meeting notes, ADRs, onboarding docs |
| Conceptual explanation, theory, research | **Memory** (reference) | 0.65 | Academic papers, design philosophy articles |
| Tool/API documentation | **Skill** (reference) | 0.75 | SDK docs, API references |
| Mixed: constraints + workflow | **Skill** + **Rule** | 0.60 | Framework guides with best practices |
| Mixed: role + workflow | **Agent** + **Skill** | 0.55 | Team lead playbooks |
| Design principle extending an existing artifact | **Integration** + **Memory** | 0.80 | Failure patterns, mental models |
| Concrete failure mode for an existing framework | **Integration** (existing rule/skill) | 0.85 | Anti-patterns, edge cases |

## Confidence scoring

```text
confidence = base_signal_score × relevance_modifier × specificity_bonus

base_signal_score  = from the Signal Pattern Table above
relevance_modifier = 1.0 (directly applicable) | 0.8 (needs adaptation) | 0.5 (tangential)
specificity_bonus  = +0.1 if content has exact file paths, commands, or code
```

**`relevance_modifier` is a single value, computed once per source-content, not per candidate type.** It measures how directly the *content itself* applies to *this project* — a property of the import, independent of which artifact type you're scoring it against. Assign it once (directly applicable / needs adaptation / tangential), then multiply every candidate type's `base_signal_score` by that same value. Varying it across candidates to nudge one type's score up is exactly the kind of self-serving miscalibration this pipeline is designed to prevent elsewhere (see the anti-gaming constraints in Gate 0) — the same discipline applies here.

## Ambiguity resolution

When the top-2 confidence scores are within 0.15 of each other:

1. Flag as ambiguous — do not auto-classify.
2. Present the top-2 candidates, their scores, and the signals supporting each.
3. Recommend a primary with rationale, but let the user decide.

## Worked examples

**Blog post → Skill.** "How to Set Up CI/CD with GitHub Actions: A Step-by-Step Guide" — step-by-step signal dominates. Skill 0.80, Rule 0.30. No ambiguity.

**Style guide → Rule.** "Always use type hints. Never use mutable defaults." — passive-constraint signal dominates. Rule 0.85, Memory 0.25. No ambiguity.

**Role description → Agent.** "The Security Reviewer focuses on OWASP Top 10, reviews PRs for vulnerabilities..." — role + behavioral profile. Agent 0.75, Skill 0.45. No ambiguity.

**Mixed content → Ambiguous.** "Our review process: always check for SQL injection (constraint), then follow this 5-step review workflow (procedure)." — both signals present. Skill 0.65, Rule 0.55 — gap 0.10 < 0.15 → flag ambiguous, present both to the user.

## Multi-artifact generation

| Scenario | Primary | Secondary | When |
|---|---|---|---|
| Tutorial with best practices | Skill | Rule (extracted constraints) | >3 constraints identified |
| Framework guide with personas | Skill | Agent (if a distinct role emerges) | Role has >3 behavioral traits |
| Research paper with actionable findings | Memory | Rule or Skill (if actionable) | Contains implementable patterns |
| Team charter with workflows | Agent | Skill (workflow portion) | Workflow has >5 steps |

## Priority order

When content could reasonably become more than one type, prefer: **Rule > Skill > Agent > Memory**. Rules are cheapest to maintain and can be enforced automatically; skills need more structure; agents are heaviest; memory is the catch-all but least actionable.

## Claim credibility (distinct from classification confidence)

Classification confidence picks WHICH artifact type to create. Claim credibility measures HOW RELIABLE the extracted claim is — a separate axis, computed from Phase 1's triangulation result:

```text
claim_credibility = authority_modifier × triangulation_modifier

authority_modifier (from the Step 1b tier):
  primary   = 1.0
  secondary = 0.8
  tertiary  = 0.5

triangulation_modifier (from Step 1c):
  multi_source_convergent = 1.0  (≥2 independent sources agree)
  multi_source_divergent  = 0.7  (independent sources disagree — needs reconciliation)
  single_source           = authority_modifier × 0.8  (capped at the source's own tier)
```

When an import carries multiple sources: if they converge, use the max per-source credibility; if they diverge, use the min and tag the extraction `[Divergent]`. This does not multiply into classification confidence — the two scores stay separate.

**Downstream usage**: credibility < 0.5 → Phase 3.5 integration denied (memory only). 0.5–0.8 → integration allowed but flagged for user review. ≥0.8 → integration eligible without extra review.

## Three different numbers named "confidence" — don't conflate them

The pipeline produces three distinct scores that are easy to blur together. Keep them separate — they measure different things and are consumed by different downstream steps:

| Score | Measures | Computed in | Consumed by |
|-------|----------|--------------|-------------|
| Extraction confidence | How faithfully a Phase 2 table row represents the source text | Phase 2 | Informal — a low score is a signal to re-read the source, nothing gates on it directly |
| Classification confidence | Which artifact type this content should become | This file, Phase 3 | The ambiguity check (gap < 0.15 → ask the user) |
| `claim_credibility` | How reliable the underlying claim is, from triangulation | This file's formula above | Phase 3.5's integration eligibility thresholds (0.5, 0.8) |

A single import can score high on one and low on another — e.g. a tertiary single-source article can have high extraction confidence (you read it accurately) and high classification confidence (it's obviously a Rule) while still having low `claim_credibility` (nothing corroborates it yet). All three gates matter independently.

## Tier boundary: organizational affiliation, not audience size or reputation

The `secondary` row requires a **named organizational affiliation** — the source is published under a company or institution's editorial umbrella, not an individual's own platform. A well-known independent practitioner writing under their own name on their own newsletter or blog — regardless of audience size or industry reputation — classifies as `tertiary` under this table, by design. Fame is not the axis; institutional accountability is.

If you believe an individual source's authority should count for more than `tertiary` in a specific case, that is exactly what `manual_override` is for: promote it explicitly with a recorded reason, rather than stretching the table's definition of `secondary` to fit. Keeping the table's boundary mechanical is what makes `pattern_matched` citations auditable across different people using this skill.
