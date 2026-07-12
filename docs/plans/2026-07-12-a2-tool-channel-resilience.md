# A2: tool-channel-resilience — ツールチャネル障害の規律集

- 作成日: 2026-07-12 / 工数見積: 0.5 人日 / リスク: 低
- 種別: ルールプラグイン

## 目的

`~/.claude/rules/workflows/tool-channel-resilience.md` を汎用化して公開する。エージェントとツール実行系の間のトランスポート障害（空応答・バッチ全滅・途中停止）に対する運用規律は、実障害から蒸留された知見であり、ドキュメントとして稀少。

## 抽出元資産

- `~/.claude/rules/workflows/tool-channel-resilience.md` — 検知シグナル + 8 規律（小バッチ / AskUserQuestion 隔離 / テキスト・ツール分離 / background+poll / edit-then-verify / 行番号一括書き込み禁止 / ポータブルコマンド / 再開時の実状態再導出）

## 公開物

1. `plugins/tool-channel-resilience/rules/tool-channel-resilience.md` — 英語化・汎用化版
2. `plugins/tool-channel-resilience/README.md` — 背景説明（どんな障害がなぜ起きるか、instructions で直せる範囲と直せない範囲）

## 実装ステップ

1. 内部固有の参照（他ルールへのリンク、`/loop` 等の内部スキル名、固有インシデント記述）を汎用表現に置換
2. 英訳。「blast radius を bound する」という設計思想（transport 障害は指示では直らない — 被害半径だけ制御できる）を README の中心命題に据える
3. macOS/BSD ポータビリティ節はそのまま活かす（`timeout` 不在・`cat -A` 不可は普遍的なハマりどころ）
4. canonical background + poll パターンのコード例を検証してから掲載（実行して RC センチネルが機能することを確認）
5. plugin.json + marketplace.json 登録

## 注意点

- 元ファイルの「See also」節は内部スキル参照なので削除または本リポジトリ内リンクに置換
- 「Correct pillar」等の Five Pillars 用語は A1（drift-patterns）へのリンクで解決する — ユニット間の内部リンクがリポジトリの回遊性を作る

## 完了の定義

- [ ] 英語版ルール + README 完成、内部参照ゼロ
- [ ] background + poll パターンの動作検証記録あり
- [ ] marketplace.json 登録済み
