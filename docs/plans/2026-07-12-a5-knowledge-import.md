# A5: knowledge-import — 外部知見の構造化取り込みスキル

- 作成日: 2026-07-12 / 工数見積: 1.5 人日 / リスク: 低〜中
- 種別: スキルプラグイン

## 目的

`~/.claude/skills/knowledge-import` を汎用化して公開する。URL・記事・外部ドキュメントから知見を抽出し、多源三角測量（triangulation）で信頼度を判定してからスキル・ルール・メモリに構造化するパイプラインは、「AI に何かを学ばせる前に裏取りさせる」という規律ごと提供する点で類例が少ない。

## 抽出元資産

- `~/.claude/skills/knowledge-import/`（SKILL.md + Gate 定義・フェーズ定義一式）

## 公開物

1. `plugins/knowledge-import/skills/knowledge-import/SKILL.md` — 英語版
2. `plugins/knowledge-import/skills/knowledge-import/references/source-routing.md` — フェッチ戦略決定表（汎用化版。専用 fetcher スキルへの誘導は「対応ツールがあれば」という条件文に置換）
3. `plugins/knowledge-import/skills/knowledge-import/references/classification-matrix.md` — 分類決定木 + claim_credibility 式
4. `plugins/knowledge-import/README.md` — パイプライン全体図（Gate 0 取得 → 抽出 → 分類 → 重複ゲート → 生成 → 検証）と三角測量の思想説明

**スコープ変更（2026-07-12、実装時判断）**:

- 元スキルの `references/triangulation-procedures.md`（citation chain 検出・org equivalence・非対話環境の詳細プロトコル）は分量が小さいため独立ファイルにせず SKILL.md 本文（Step 1c・Gate 0）へ inline 統合。ファイル数を絞りナビゲーションを単純化
- `references/remediation-procedures.md` は「2026-06-22 セッション由来」という内部の作業日記的な導入部を削り、Step A/B/C の手続き自体は既に SKILL.md 本文に inline されているため独立ファイルとして複製せず
- `references/artifact-templates.md`（Phase 7 出力レポートのテンプレート）と Phase 7「Registration」節（`docs/guides/skills-agents-commands/` への登録）はこの環境固有の運用ディレクトリ構造に強く結合しており汎用化の価値が低いため**非同梱**。公開版は Phase 6（Validation Gates）で完結させる
- 日本語強制ゲート（Gate 4 の frontmatter 日本語要件等）は本リポジトリの CLAUDE.md 規約であり公開スキルの要件ではないため「プロジェクトの既存規約に合わせる」という汎用文言に置換
- `${CLAUDE_SKILL_DIR}` というこの環境固有と思われる変数参照は排し、スキルディレクトリからの相対参照に統一
- `知識インポートアナリスト`（Companion Agent 節）は本リポジトリ固有のカスタムエージェントで同梱対象外のため削除
- `CLAUDE_BATCH_MODE` という環境固有らしい変数名の断定を避け、「非対話環境（CI/バッチ、ユーザー入力待ちができない状況）」という一般的な条件記述に置換

## 実装ステップ

1. 現行スキルのフェーズ構成を棚卸しし、内部依存を特定する（登録先パスが `~/.claude/` 固定になっている箇所、内部メモリ規約への依存、他スキルへの誘導）
2. 登録先をプロジェクト相対（`.claude/skills/` 等）に切り替え可能にする — 公開版はユーザーの環境差を吸収する必要がある
3. 出所ラベル体系（primary/secondary/tertiary、Route A/B）は本スキルの核なので保持し、README で体系ごと説明する
4. 英訳。フェーズ数が多いので Progressive Disclosure（SKILL.md は薄く、詳細は参照ファイルへ）を維持する
5. 実走検証: 公開されている技術記事 1 本を実際に取り込み、生成物（ルール or スキル）が Gate を通過するか確認
6. plugin.json + marketplace.json 登録

## 注意点

- 9 ユニット中で最も内部結合が強い候補 — 汎用化で機能を削りすぎると差別化点（Gate 群）が消える。Gate はすべて残し、登録先だけ差し替える方針
- 英語圏では "knowledge distillation for agent configs" のような説明の方が通じる — README の言葉選びは英語ネイティブの検索語に寄せる

## 完了の定義

- [x] 英語版 SKILL.md + 参照ファイル完成、登録先が可搬化されている（grep スキャン CLEAN、`claude plugin validate` PASS）
- [ ] 外部記事相当のコンテンツでの実走検証記録あり（エージェント実行中）
- [x] marketplace.json 登録済み
