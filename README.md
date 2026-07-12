# agent-harness-lab

Small, focused plugins for AI agent harness reliability — drift-pattern countermeasures, debugging discipline, and adversarial verification for Claude Code and compatible coding agents.

> **Status**: pre-release. Units are published incrementally — see [docs/plans/](docs/plans/2026-07-12-roadmap.md) for the roadmap. This README will be replaced by the Drift Patterns catalog (unit A1) at first public release.

## Planned units

| Unit | What it does |
|------|--------------|
| drift-patterns | Catalog of 11 agent failure modes and their harness countermeasures |
| tool-channel-resilience | Discipline rules for surviving tool-transport failures |
| systematic-debugging | Phased debugging skill with question-reframing after repeated failure |
| knowledge-import | Triangulated pipeline for importing external knowledge into agent configs |
| empirical-prompt-tuning | Bias-free prompt improvement via fresh-subagent evaluation |
| voicevox-agent-notify | Voice status notifications for agent sessions (Japanese-first) |
| step-back | Hook that detects error-repetition loops and forces premise re-evaluation |
| hook-testkit | Unit-test harness for Claude Code hooks |
| game-theory-reviewer | Builder-vs-Breaker adversarial review framework |

## Install (once released)

```text
/plugin marketplace add hiro178/agent-harness-lab
```

## License

MIT
