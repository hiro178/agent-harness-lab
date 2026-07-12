# B4: hook-testkit — Claude Code フック単体テストハーネス

- 作成日: 2026-07-12 / 工数見積: 2 人日 / リスク: 低
- 種別: CLI ツール（新規開発）

## 目的

Claude Code のフックを書く人全員が潜在ユーザーなのに、フックの単体テスト手段が存在しない空白を埋める。フックイベントの JSON フィクスチャを生成し、フックスクリプトに流し込み、exit code と stdout（decision / additionalContext / systemMessage）をアサーションする。

## 設計概要

```text
hook-testkit/
├── fixtures/                 # イベント種別ごとの標準 JSON フィクスチャ
│   ├── pre-tool-use/         # Bash, Edit, Write, ... の代表入力
│   ├── post-tool-use/
│   ├── session-start/
│   └── ...（全フックイベント網羅）
├── bin/hook-test             # ランナー本体
└── examples/                 # テスト定義例
```

テスト定義（YAML or シェル）:

```yaml
- name: blocks rm -rf
  hook: ./hooks/deny-dangerous.sh
  fixture: pre-tool-use/bash
  override: { tool_input: { command: "rm -rf /" } }
  expect: { exit: 2, stderr_contains: "blocked" }
```

- 実装言語は bash + jq（フック実行環境と同一依存、インストール障壁ゼロ）を第一候補とする。YAML パースが重ければテスト定義を JSON にする
- スナップショットモード: 期待値未定義時は初回実行結果を記録し、以後の差分を検出

## 実装ステップ

1. Claude Code 公式ドキュメントから全フックイベントの入力スキーマを収集し、フィクスチャを整備する（スキーマの正確性がこのツールの生命線 — 出典バージョンを各フィクスチャに記録）
2. ランナー実装: フィクスチャ + override 合成 → フック起動 → exit / stdout / stderr アサーション
3. 自己テスト: ランナー自体のテストを同梱（テストツールがテストされていないのは説得力を欠く）
4. 実利用検証: 手持ちの既存フック 2〜3 本に対してテストを書き、実際に 1 件でも仕様齟齬を発見できたらその事例を README に載せる
5. CI レシピ（GitHub Actions で hook-test を回す workflow 例）を examples に同梱
6. plugin.json + marketplace.json 登録（CLI だがプラグイン形式でスキル `/hook-test` からも起動可能にする）

## 注意点

- フックイベントスキーマは Claude Code のバージョンで変わり得る — フィクスチャに `schema_version` を持たせ、追従を Issue 駆動にする運用を README に明記
- 既存 OSS に直接競合が見当たらないことを公開前に再確認する（公開時点の調査記録を残す）

## 完了の定義

- [ ] 主要フックイベントのフィクスチャ + ランナー + 自己テスト完成
- [ ] 既存フックでの実利用検証記録（発見事例があれば README 掲載）
- [ ] CI レシピ同梱
- [ ] marketplace.json 登録済み
