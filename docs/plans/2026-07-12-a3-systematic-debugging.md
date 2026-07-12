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

**スコープ変更（2026-07-12、実装時判断）**: 元スキルの補助ファイル 4 本（root-cause-tracing / defense-in-depth / condition-based-waiting / find-polluter.sh）は obra/superpowers（MIT）からの直輸入と確認。再配布しても差別化ゼロで「コピー」批判リスクのみのため**同梱しない**。SKILL.md は 4 フェーズ核を superpowers 帰属と明記した上で、独自拡張（Phase 3.5 分類タクソノミー / 3.6 STOP_FOR_HUMAN / 3.7 構造化出力 / 5 問い再構成）を主役とする「agent-oriented extension」として位置づける。

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

- [x] 英語版 SKILL.md 完成、内部参照ゼロ（grep スキャン CLEAN、`claude plugin validate` PASS）
- [x] 実走検証 1 本（Phase 5 発火ケース含む）の記録あり
- [x] marketplace.json 登録済み
- [ ] この時点で 3 ユニット公開 → awesome 系リストへの掲載 PR 提出（ユーザー確認待ち）

## 検証記録（2026-07-12）

公開版 SKILL.md のみを読ませた新規 subagent 2 体で実走検証（バイアス排除のため本セッションの文脈は与えない）:

**ケース 1 — 実バグ診断**: `tmp/verify-a3/` に可変デフォルト引数バグ（`def add_tag(tag, tags=[])`）を仕込み、テスト失敗から診断させた。結果 PASS — Phase 1〜3.7 を順守（再現 → データフロー追跡 → 単一仮説を introspection で確定 → LOGIC_ERROR 分類 → STOP_FOR_HUMAN 5 条件の非該当判定 → 構造化出力 NEXT_ACTION: FIX / CONFIDENCE: HIGH）。編集禁止の制約下で FIX_PLAN のみ提示し、勝手な修正なし。

**ケース 2 — Phase 5 発火**: 「3 回の修正がすべて局所的には成功（計 27 分削減）だが目標（300 分→60 分未満）に遠い」シナリオで Phase 5 が正しく発火。個別最適化 vs 構造的排除の算術を適用し、NEXT_ACTION: REFRAME・TASK_DECOMPOSITION_PROBLEM 分類の構造化出力を生成。

**検証からのフィードバック 3 件を SKILL.md に反映済み**:

1. Phase 5 の「failed」定義を明文化（局所成功でも目標未達なら失敗にカウント）
2. Phase 3.5 に分類ノート追加（設計起因の性能問題は TASK_DECOMPOSITION_PROBLEM）
3. Phase 2 に参照実装が無い場合のフォールバック（言語標準イディオムとの比較）を追加
