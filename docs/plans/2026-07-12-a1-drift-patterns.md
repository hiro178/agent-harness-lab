# A1: drift-patterns — エージェント失敗モードカタログ

- 作成日: 2026-07-12 / 工数見積: 1 人日 / リスク: 低
- 種別: ドキュメント + ルールプラグイン（旗艦コンテンツ）

## 目的

`~/.claude/rules/review/harness-engineering.md` の Drift Patterns 表（10 種のエージェント失敗モードと対策。2026-07-12 実測: 148〜157 行目）を独立コンテンツとして切り出し、本リポジトリの README 旗艦に据える。Verifier Theater・Phantom Confabulation・Emergent menu drift は他 OSS に類例がなく、最大の差別化資産。

## 抽出元資産

- `~/.claude/rules/review/harness-engineering.md` — Drift Patterns 表（10 行）、Five Pillars、Autonomy Rollout Ladder
- 各パターンの出典メモリ（`reference_*`）— 公開時は外部ソース名に展開する

## 公開物

1. `README.md`（リポジトリ直下）— 10 パターンの一覧表 + 各パターンへのリンク。英語
2. `patterns/<slug>.md` × 10 — 症状 / 実例 / ハーネス対策 / 出典。英語
3. `plugins/drift-patterns/` — 対策をスキル化した Claude Code プラグイン（`skills/drift-patterns/SKILL.md`。プラグイン機構は `rules/` コンポーネントを持たないため、オンデマンドロードされる skill として配布する — 2026-07-12 計画変更）

## 実装ステップ

1. Drift Patterns 表を英訳し、内部参照（`mem:` / `reference_*` / 内部ファイルパス）をすべて外部出典表記に置換する
2. 各パターンを 1 ファイルに分割し、「Symptom → Real-world example → Countermeasure → Sources」の 4 節構成で書く
3. 出所ラベルを付与する: LayerX Fukushima（2026-04）、Rajasekaran/Anthropic（2026-03）、aidlc-workflows（AWS Labs, MIT-0）、Seino/Classmethod + addyosmani（Verifier Theater）、arXiv 2306.05499 / 2503.16248（Phantom Confabulation の機構）。独自観察（一次経験由来の記述）は "observed in our sessions" と明示し、輸入知見と区別する
4. プラグイン化: カタログ + 対策を `plugins/drift-patterns/skills/drift-patterns/SKILL.md` に配置し `plugin.json` を書く
5. marketplace.json に登録

## 注意点

- 内部リポジトリ（Claude-Code-Configure）の非公開情報（パス・メモリ名・インシデント詳細）を持ち込まない
- Phantom Confabulation の named framing は tertiary ソース由来 — 断定でなく「we call this pattern」の一人称スタンスで書く（出所規律）
- ライセンス整合: aidlc-workflows 由来部分は MIT-0 なので MIT 配布に問題なし

## 完了の定義

- [ ] 10 パターンすべて英語ドキュメント化、内部参照ゼロ
- [ ] 出典が全パターンに付与されている
- [ ] プラグインとして `/plugin install drift-patterns@agent-harness-lab` が通る
- [ ] この完成をもってリポジトリを public 化
