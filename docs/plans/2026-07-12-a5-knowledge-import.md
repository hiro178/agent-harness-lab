# A5: knowledge-import — 外部知見の構造化取り込みスキル

- 作成日: 2026-07-12 / 工数見積: 1.5 人日 / リスク: 低〜中
- 種別: スキルプラグイン

## 目的

`~/.claude/skills/knowledge-import` を汎用化して公開する。URL・記事・外部ドキュメントから知見を抽出し、多源三角測量（triangulation）で信頼度を判定してからスキル・ルール・メモリに構造化するパイプラインは、「AI に何かを学ばせる前に裏取りさせる」という規律ごと提供する点で類例が少ない。

## 抽出元資産

- `~/.claude/skills/knowledge-import/`（SKILL.md + Gate 定義・フェーズ定義一式）

## 公開物

1. `plugins/knowledge-import/skills/knowledge-import/SKILL.md` — 英語版
2. `plugins/knowledge-import/README.md` — パイプライン全体図（Gate 0 取得 → 抽出 → 分類 → 重複ゲート → 生成 → 検証 → 登録）と三角測量の思想説明

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

- [ ] 英語版 SKILL.md + 参照ファイル完成、登録先が可搬化されている
- [ ] 外部記事 1 本での実走検証記録あり
- [ ] marketplace.json 登録済み
