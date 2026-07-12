# A6: empirical-prompt-tuning — 実証的プロンプト改善スキル

- 作成日: 2026-07-12 / 工数見積: 1 人日 / リスク: 低
- 種別: スキルプラグイン

## 目的

`~/.claude/skills/empirical-prompt-tuning` を汎用化して公開する。プロンプト・スキル・エージェント指示を「書いた本人が評価する」のではなく、バイアスのない新規 subagent に実行させて両面評価（実行側の自己申告 + 指示側メトリクス）し、収束まで反復する。Drift Patterns の Quality self-overconfidence（自己採点ループ）への実装レベルの対策であり、A1 と直結する。

## 抽出元資産

- `~/.claude/skills/empirical-prompt-tuning/`（SKILL.md + 評価手順一式）

## 公開物

1. `plugins/empirical-prompt-tuning/skills/empirical-prompt-tuning/SKILL.md` — 英語版
2. `plugins/empirical-prompt-tuning/README.md` — Generator/Evaluator 分離の思想と、A1 drift-patterns（Quality self-overconfidence 行）へのクロスリンク

## 実装ステップ

1. 現行スキルの評価プロトコル（新規 subagent 派遣・両面評価・収束条件）を棚卸しし、内部依存を特定
2. 英訳 + 汎用化。subagent 派遣は Claude Code 標準の Agent 機構のみに依存させる（カスタムエージェント前提を外す）
3. 実走検証: 意図的に曖昧なサンプルプロンプトを 1 本用意し、改善ループが収束するまで回した記録を取る（before/after のプロンプト差分と評価スコアを残す）
4. その記録を README の「Worked example」節としてそのまま掲載する — 実証系スキルは実証記録が最良のマーケティング
5. plugin.json + marketplace.json 登録

## 注意点

- 「収束条件」が主観的にならないよう、公開版では反復上限（例: 5 回）と停止基準を明文化する
- A1 とのクロスリンクを双方向に張る（drift-patterns 側の対策列からも本プラグインを参照）— リポジトリ内回遊がスター転換率を上げる

## 完了の定義

- [ ] 英語版 SKILL.md 完成、内部依存ゼロ
- [ ] Worked example（実走記録）が README に掲載されている
- [ ] marketplace.json 登録済み
