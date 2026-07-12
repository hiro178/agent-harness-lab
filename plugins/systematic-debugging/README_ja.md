# systematic-debugging plugin

*[English](README.md) | [日本語](README_ja.md)*

根本原因優先のデバッグを強制可能な規律として — エージェントワークフロー向けに拡張。

## よく知られたバージョンとの違い

4フェーズの核（調査 → パターン照合 → 仮説 → 実装）は [obra/superpowers](https://github.com/obra/superpowers)（MIT）のシステマティックデバッグの流儀に従っており、スキル内で帰属を明記している。本プラグインは自律・準自律ワークフローに必要な4つの**エージェント指向拡張**を追加する:

| 拡張 | 追加内容 |
|-----------|--------------|
| **Phase 3.5 — 根本原因分類** | 10 分類のタクソノミー（`LOGIC_ERROR`、`CONFIG_GAP`、`SPEC_CONFLICT` など）により、セッションやエージェントをまたいでデバッグ出力を機械比較可能にする |
| **Phase 3.6 — STOP_FOR_HUMAN プロトコル** | 事前定義されたエスカレーション条件（仕様衝突・認証情報の境界・公開 API の破壊）と固定のハンドオフ形式 — エージェントは*いつ独断で決めてはいけないか*を知る |
| **Phase 3.7 — 構造化出力** | 固定のレポート形式（`ROOT_CAUSE` / `CATEGORY` / `FIX_PLAN` / `VERIFICATION` / `NEXT_ACTION` / `CONFIDENCE`）により、親エージェントやオーケストレーターがデバッグ結果をプログラム的に消費できる |
| **Phase 5 — 問いの再構成** | 修正試行が3回失敗したら、答えのデバッグをやめて問いをデバッグする: 「どう X を速くするか」→「そもそも X をすべきか」。構造的排除は個別最適化に勝る |

## インストール

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install systematic-debugging@agent-harness-lab
```

## 発動タイミング

バグ・テスト失敗・予期しない挙動に遭遇し、修正を提案する前にオンデマンドで発動する。時間的プレッシャー下で「明らかな」応急処置が手元にある——まさに従いたくない瞬間にこそ最も価値を発揮するよう設計されている。
