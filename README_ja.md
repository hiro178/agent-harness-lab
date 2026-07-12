# agent-harness-lab

*[English](README.md) | [日本語](README_ja.md)*

**AI コーディングエージェントが静かに失敗する仕組みのフィールドカタログ — そして、それを止めるハーネス対策集。**

エージェントは滅多に派手には失敗しない。早すぎる「完了」宣言、自分の宿題の自己採点、テストが通るまでテストを弱める、時には存在しない攻撃者を幻視することさえある。以下の各パターンは実際のエージェントハーネス運用から蒸留し、公開されているソースと突き合わせて検証したものだ。それぞれ失敗の名前・外から見える兆候・**構造的に**強制できる対策（プロンプトではなくハーネスで強制する対策）を示す。

> *ハーネス*: モデルの周りにあるすべて — フック、パーミッション、ワークフロー、検証ゲート。モデルは知性を提供し、ハーネスはそれを信頼できるものにする（用語は Mitchell Hashimoto が 2026-02 に提唱、OpenAI の "Harness Engineering" で定着）。

## 10 の Drift Pattern（失敗モード）

| # | パターン | 一言で言う症状 | 対策の分類 |
|---|---------|-----------------|---------------------|
| 1 | [Completion Misidentification](patterns/completion-misidentification.md) | 早すぎる「完了」宣言 | 外部完了ゲート |
| 2 | [Quality Self-Overconfidence](patterns/quality-self-overconfidence.md) | 自分の出力を自分で採点する | Generator/Evaluator 分離 |
| 3 | [Cumulative Deviation](patterns/cumulative-deviation.md) | 小さなズレが 10 ステップ以上かけて蓄積する | 定期的な仕様準拠チェックポイント |
| 4 | [Goal Drift](patterns/goal-drift.md) | 圧縮（compaction）後に制約が消える | 制約の再注入、独立したコンテキストウィンドウ |
| 5 | [Functional Stubs](patterns/functional-stubs.md) | UI は存在するがハンドラがモック — テストは緑のまま | ライブナビゲーション評価 |
| 6 | [Step-Skip Rationalization](patterns/step-skip-rationalization.md) | 検証をスキップするもっともらしい言い訳 | アンチ合理化テーブル |
| 7 | [Context Pollution Cascade](patterns/context-pollution-cascade.md) | 1エージェントの誤りがパイプライン下流で増幅する | コンテキスト所有契約 |
| 8 | [Emergent Menu Drift](patterns/emergent-menu-drift.md) | ワークフローが定義していない選択肢を勝手に作る | ステージテンプレートのハードコード |
| 9 | [Verifier Theater](patterns/verifier-theater.md) | テストを弱めて作業を「合格」させる | 検証者はオラクル（テスト）を編集できない |
| 10 | [Phantom Confabulation](patterns/phantom-confabulation.md) | 起きていない攻撃や対話を報告する | ロールベースのトランスクリプト・フォレンジック |

### 設計への含意

自律エージェントセッションの 10〜20% はこれらのいずれかの形で誤ると想定せよ。制約を緩めることでも、エージェント自身の説明を信じることでもなく、**並列実行と構造的ゲート**で補償する。

### 5 つのハーネス原則（Five Pillars）

上記の対策は 5 つの繰り返し現れる仕組みに対応する: **Constrain**（パーミッションでスコープを制限する）、**Inform**（長いコンテキストより密なコンテキスト）、**Verify**（チェックを提案でなく制御フローにする）、**Correct**（構造的なエラー処理）、**Prune**（モデルの能力向上に合わせて古い足場を外す）。核となる原則: *「エージェントにテストを実行するよう指示する」のではなく「エージェントの判断に関わらずワークフローがテストを実行する」ようにする*。

## Claude Code プラグインとしてインストール

```text
/plugin marketplace add hiro178/agent-harness-lab
/plugin install drift-patterns@agent-harness-lab
```

### 公開済みユニット

| プラグイン | 提供内容 |
|--------|-------------------|
| [`drift-patterns`](plugins/drift-patterns/) | 上記カタログをオンデマンドスキル化: 診断テーブル・対策・設計時チェックリスト |
| [`tool-channel-resilience`](plugins/tool-channel-resilience/) | ツールチャネル劣化時の規律: 小バッチ・background+poll・edit-then-verify |
| [`systematic-debugging`](plugins/systematic-debugging/) | エージェント拡張版の根本原因優先デバッグ: 原因タクソノミー・STOP_FOR_HUMAN エスカレーション・構造化出力・問い再構成 |
| [`knowledge-import`](plugins/knowledge-import/) | 三角測量ゲート付きパイプライン: 外部コンテンツは出所検証・重複排除・検証ゲートを通過して初めて生成に到達する |

## 今後のユニット（順次公開）

このリポジトリは小粒で焦点を絞ったハーネス信頼性ユニットを継続的に公開する — ツールチャネル障害の規律、体系的デバッグ、フック単体テスト、敵対的 Builder-vs-Breaker レビューなど。ロードマップは [docs/plans/](docs/plans/2026-07-12-roadmap.md) を参照。

## 出典・帰属

本カタログは独自の運用観察と公開済みの知見を統合したもので、パターンごとに出典を明記している: Fukushima (LayerX, 2026-04)、Rajasekaran (Anthropic, 2026-03)、Thariq & Sid Bidasaria (Anthropic, 2026-06)、AWS Labs aidlc-workflows (MIT-0)、Addy Osmani、Block Engineering、Seino (Classmethod)、Steinberger (OpenClaw)、AL-Awady、arXiv:2306.05499 / arXiv:2503.16248。独自成果として明記した行は、長時間稼働するエージェントセッションからの一次観察である。

## ライセンス

MIT
