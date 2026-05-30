---
description: 現在の機能仕様で記述不足の箇所を見つけ、最大 5 件の高精度な確認質問を行い、その回答を spec に反映する。
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## Outline

目的: アクティブな機能仕様内の曖昧さや未決定事項を特定・削減し、明確化内容を spec ファイルへ直接反映する。

注記: この明確化フローは `/speckit.plan` の前に実行し、完了していることが期待される。ユーザーが明示的に明確化を省略する（例: exploratory spike）場合は進めてよいが、後工程の手戻りリスクが上がることを警告すること。

実行手順 (Execution steps):

1. リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly` を **一度だけ** 実行する。JSON から最低限次を読む。
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - 必要なら `IMPL_PLAN`, `TASKS`
   - JSON 解析に失敗したら中断し、`/speckit.specify` の再実行または feature branch 環境確認を促す
   - 引数中のシングルクォートは必要に応じてエスケープする

2. 現在の spec ファイルを読み、次の分類で曖昧さ / カバレッジを評価する。各カテゴリの状態は `Clear / Partial / Missing` とする。優先順位付けに使う内部カバレッジマップを作るが、質問がなければその場合のみ出力してよい。

   機能範囲と挙動 (Functional Scope & Behavior):
   - 主要なユーザーゴールと成功基準 (Core user goals & success criteria)
   - 明示的なスコープ外の宣言 (Explicit out-of-scope declarations)
   - ユーザーロール / ペルソナの差別化 (User roles / personas differentiation)

   ドメインとデータモデル (Domain & Data Model):
   - エンティティ、属性、関係性 (Entities, attributes, relationships)
   - 識別性と一意性のルール (Identity & uniqueness rules)
   - ライフサイクル / 状態遷移 (Lifecycle / state transitions)
   - データ量 / スケールの想定 (Data volume / scale assumptions)

   インタラクションとUXフロー (Interaction & UX Flow):
   - 重要なユーザージャーニー / シーケンス (Critical user journeys / sequences)
   - エラー / 空 / ローディング状態 (Error / empty / loading states)
   - アクセシビリティまたはローカライズに関するメモ (Accessibility or localization notes)

   非機能品質属性 (Non-Functional Quality Attributes):
   - パフォーマンス (Performance)
   - 拡張性 (Scalability)
   - 信頼性と可用性 (Reliability & availability)
   - 観測容易性 (Observability)
   - セキュリティとプライバシー (Security & privacy)
   - コンプライアンス / 規制上の制約 (Compliance / regulatory constraints)

   統合と外部依存関係 (Integration & External Dependencies):
   - 外部サービス / API およびその失敗モード (External services / APIs and failure modes)
   - データインポート / エクスポートの形式 (Data import / export formats)
   - プロトコル / バージョニングの想定 (Protocol / versioning assumptions)

   エッジケースと異常系ハンドリング (Edge Cases & Failure Handling):
   - ネガティブシナリオ (Negative scenarios)
   - レート制限 / スロットリング (Rate limiting / throttling)
   - 競合解決 (Conflict resolution)

   制約とトレードオフ (Constraints & Tradeoffs):
   - 技術的制約 (Technical constraints)
   - 明示的なトレードオフまたは却下された代替案 (Explicit tradeoffs or rejected alternatives)

   用語と一貫性 (Terminology & Consistency):
   - 標準的な用語集の用語 (Canonical glossary terms)
   - 回避すべき同義語 / 非推奨の用語 (Avoided synonyms / deprecated terms)

   完了シグナル (Completion Signals):
   - 受け入れ基準のテスト容易性 (Acceptance criteria testability)
   - 測定可能な Definition of Done（完了の定義）スタイルの指標 (Measurable Definition of Done style indicators)

   その他 / プレースホルダー (Misc / Placeholders):
   - TODO マーカー / 未解決の決定事項 (TODO markers / unresolved decisions)
   - 曖昧語（`robust`、`intuitive` など）

   `Partial (一部あり)` または `Missing (欠落)` のカテゴリごとに質問候補を追加する。ただし次の場合は除外する:
   - 実装や検証戦略に実質的な影響がない
   - 設計 (planning) フェーズで扱う方が適切

3. 質問候補キューを内部的に優先順位付けして作成する（最大 5 件）。一度にすべてを提示しないこと。制約は以下の通り。
   - セッション全体で最大 10 問
   - 各質問は次のどちらかで答えられること
     - 2〜5 個の相互排他的な短い選択肢
     - 5語/15文字以内の短答
   - 回答がアーキテクチャ、データモデリング、タスク分解、テスト設計、UXの挙動、運用の準備状態 (operational readiness)、コンプライアンス検証に実質影響するものだけを含める
   - 高影響な未解決カテゴリを優先し、低影響な質問を連続させない
   - 既回答事項、単なる好み、正しさを妨げない設計 (plan) の詳細は除外
   - 手戻り削減や受け入れテストのズレ防止に効くものを優先
   - 未解決が 5 カテゴリ超なら `(影響度 * 不確実性) (Impact * Uncertainty)` で上位 5 件に絞る

4. **順次質問ループ (Sequential questioning loop)**:
   - 質問は **必ず 1 問ずつ** 出す
   - 選択肢形式の質問では:
     - 全選択肢を評価し、最適案を 1 つ推薦する
     - 推薦理由を 1〜2 文で示す
     - `**Recommended:** Option [X] - <理由>` の形式にする
     - その後、以下形式の表を出す

       | 選択肢 (Option) | 説明 (Description) |
       |--------|-------------|
       | A | <選択肢Aの説明> |
       | B | <選択肢Bの説明> |
       | C | <選択肢Cの説明> |
       | Short | 別の短い回答を入力（5語/15文字以内） |

     - 続けて `You can reply with the option letter...` (選択肢のアルファベットで回答するか、別の短い回答を入力してください) を表示する

   - 短答形式では:
     - 文脈に基づいた推奨回答を提示する
     - `**Suggested:** <推奨回答> - <理由>` の形式にする
     - その後に短答フォーマット案内を行う

   - ユーザー回答後:
     - `yes`、`recommended`、`suggested`、`はい`、`推奨`、`提案` なら推薦 / 提案値を採用
     - それ以外は選択肢または 5語/15文字以内制約へ適合するか確認
     - 曖昧なら同一質問内で再確認する
     - 問題なければ作業メモリ (session memory) に記録し、次の質問へ進む

   - 次の場合は質問を止める:
     - 重要な曖昧さが早期に解消した
     - ユーザーが `done`、`good`、`no more`、`完了`、`終了`、`十分` など完了意思を示した
     - 5 問に達した

   - 先の質問候補を先出ししてはならない
   - 最初から有効な質問がなければ、重要な曖昧さはないと即時報告する

5. **回答受託ごとの統合 (Integration after EACH accepted answer)**:
   - 読み込んだ spec とその生内容をインメモリ (in-memory) で保持する
   - セッション最初の反映時に `## Clarifications` セクションがなければ追加する
   - その下に今日の日付で `### Session YYYY-MM-DD` を作る
   - 受理後すぐに `- Q: <質問> -> A: <最終回答>` を追記する
   - さらに、内容に応じて最も適切なセクションへ即座に反映する
     - 機能の曖昧さ (Functional ambiguity) -> 機能要件 (Functional Requirements)
     - ユーザーインタラクション/アクターの区別 (User interaction / actor distinction) -> ユーザーストーリー/アクター (User Stories / Actors)
     - データの形状/エンティティ (Data shape / entities) -> データモデル (Data Model)
     - 非機能の制約 (Non-functional constraint) -> 非機能/品質属性 (Non-Functional / Quality Attributes)
     - エッジケース/異常系フロー (Edge case / negative flow) -> エッジケース/エラーハンドリング (Edge Cases / Error Handling)
     - 用語の衝突 (Terminology conflict) -> 用語の正規化/用語集 (Glossary)
   - 以前の曖昧文が無効になる場合は重複追記ではなく置換する
   - 各反映後に spec を保存してコンテキスト喪失を防ぐ
   - 見出し階層は保ち、無関係な並び替えはしない
   - 追加文は最小限かつ検証可能にする

6. **検証 (Validation)**（各書き込み後と最終時）
   - Clarifications セッションには受理回答ごとにちょうど 1 行あること
   - 受理質問数は 5 以下であること
   - 反映後のセクションに、解消対象だった曖昧語が残っていないこと
   - 無効化された古い記述が残っていないこと
   - Markdown構造が妥当で、新規見出しは `## Clarifications` と `### Session YYYY-MM-DD` のみであること
   - 用語の一貫性が保たれていること

7. 更新済み spec を `FEATURE_SPEC` に書き戻す。

8. **完了レポート (Report completion)**:
   - 質問数と回答数
   - 更新した spec のパス
   - 触ったセクション一覧
   - 各カテゴリを `解決済み (Resolved) / 保留 (Deferred) / 明確 (Clear) / 未解決 (Outstanding)` で示すカバレッジ表
   - Outstanding / Deferred があれば、`/speckit.plan` に進むべきか、後で `/speckit.clarify` を再実行すべきか推奨する
   - 推奨次コマンド

動作ルール (Behavior rules):

- 重要な曖昧さが見つからなければ `No critical ambiguities detected worth formal clarification. (正式な確認が必要な重大な曖昧さは検出されませんでした。)` と返し、次へ進むことを勧める
- spec がなければ `/speckit.specify` を先に実行するよう案内する
- 新規質問は 5 問を超えない
- 機能的明確さを妨げない限り、技術スタックの推測質問は避ける
- `stop`、`done`、`proceed` などの早期終了意思を尊重する
- 質問不要ならコンパクトな全 Clear サマリを出して先へ促す
- 上限到達時に高影響未解決が残れば、理由付きで 保留 (Deferred) へ明示する

Context for prioritization: $ARGUMENTS
