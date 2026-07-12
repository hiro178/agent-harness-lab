# A3: systematic-debugging — 体系的デバッグスキル

- 作成日: 2026-07-12 / 工数見積: 1 人日 / リスク: 低
- 種別: スキルプラグイン

## 目的

`~/.claude/skills/systematic-debugging` を英語化・汎用化して公開する。差別化点は Phase 5「3 回失敗したら問いを再構成する」— 「どう X を速くするか」→「そもそも X をすべきか」への転換で、個別最適化（N 回の操作を高速化）より構造的排除（(1-p)×N 回の操作を丸ごと消す）を優先する判断枠組み。

## 抽出元資産

- `~/.claude/skills/systematic-debugging/`（SKILL.md + 参照ファイル一式）

## 公開物

1. `plugins/systematic-debugging/skills/systematic-debugging/SKILL.md` — 英語版
2. `plugins/systematic-debugging/README.md` — Phase 構成の概観と Phase 5 の思想説明

## 実装ステップ

1. 現行 SKILL.md の全 Phase を棚卸しし、内部依存（他スキル参照・日本語出力ルール前提）を特定する
2. 英訳 + 汎用化。`test-driven-development` 等の内部スキルへの誘導は「後続プラグインで提供予定」の注記か削除で処理
3. frontmatter（name / description / effort）を plugin 配布形式に合わせて調整
4. 新規セッションで実際にスキルを起動し、Phase 進行が英語版でも機能するか 1 ケース検証（意図的にバグを仕込んだ小スクリプトで実走）
5. plugin.json + marketplace.json 登録

## 注意点

- 「What Didn't Work テーブル」（失敗アプローチの記録様式）は experiment-driven-development 由来 — 該当節を自己完結化する
- 検証は読むだけでは不可。実走 1 本（敵対ケース: 3 回失敗を意図的に発生させ Phase 5 の再構成が発火するか）を完了条件に含める

## 完了の定義

- [ ] 英語版 SKILL.md 完成、内部参照ゼロ
- [ ] 実走検証 1 本（Phase 5 発火ケース含む）の記録あり
- [ ] marketplace.json 登録済み
- [ ] この時点で 3 ユニット公開 → awesome 系リストへの掲載 PR 提出
