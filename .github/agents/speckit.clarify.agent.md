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

Execution steps:

1. リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly` を **一度だけ** 実行する。JSON から最低限次を読む。
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - 必要なら `IMPL_PLAN`, `TASKS`
   - JSON 解析に失敗したら中断し、`/speckit.specify` の再実行または feature branch 環境確認を促す
   - 引数中のシングルクォートは必要に応じてエスケープする

2. 現在の spec ファイルを読み、次の分類で曖昧さ / カバレッジを評価する。各カテゴリの状態は `Clear / Partial / Missing` とする。優先順位付けに使う内部カバレッジマップを作るが、質問がなければその場合のみ出力してよい。

   Functional Scope & Behavior:
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   Domain & Data Model:
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle / state transitions
   - Data volume / scale assumptions

   Interaction & UX Flow:
   - Critical user journeys / sequences
   - Error / empty / loading states
   - Accessibility or localization notes

   Non-Functional Quality Attributes:
   - Performance
   - Scalability
   - Reliability & availability
   - Observability
   - Security & privacy
   - Compliance / regulatory constraints

   Integration & External Dependencies:
   - External services / APIs and failure modes
   - Data import / export formats
   - Protocol / versioning assumptions

   Edge Cases & Failure Handling:
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution

   Constraints & Tradeoffs:
   - Technical constraints
   - Explicit tradeoffs or rejected alternatives

   Terminology & Consistency:
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   Completion Signals:
   - Acceptance criteria testability
   - Measurable Definition of Done style indicators

   Misc / Placeholders:
   - TODO markers / unresolved decisions
   - 曖昧語（`robust`, `intuitive` など）

   `Partial` または `Missing` のカテゴリごとに質問候補を追加する。ただし次の場合は除外する:
   - 実装や検証戦略に実質的な影響がない
   - planning フェーズで扱う方が適切

3. 質問候補キューを内部的に優先順位付けして作る（最大 5 件）。一度に全部は出さない。制約は次の通り。
   - セッション全体で最大 10 問
   - 各質問は次のどちらかで答えられること
     - 2〜5 個の相互排他的な短い選択肢
     - 5 語以内の短答
   - 回答が architecture、data modeling、task decomposition、test design、UX behavior、operational readiness、compliance validation に実質影響するものだけを含める
   - 高影響な未解決カテゴリを優先し、低影響な質問を連続させない
   - 既回答事項、単なる好み、正しさを妨げない plan 詳細は除外
   - 手戻り削減や受け入れテストのズレ防止に効くものを優先
   - 未解決が 5 カテゴリ超なら `(Impact * Uncertainty)` で上位 5 件に絞る

4. **Sequential questioning loop**:
   - 質問は **必ず 1 問ずつ** 出す
   - 選択肢形式の質問では:
     - 全選択肢を評価し、最適案を 1 つ推薦する
     - 推薦理由を 1〜2 文で示す
     - `**Recommended:** Option [X] - <reasoning>` の形式にする
     - その後、以下形式の表を出す

       | Option | Description |
       |--------|-------------|
       | A | <Option A description> |
       | B | <Option B description> |
       | C | <Option C description> |
       | Short | Provide a different short answer (<=5 words) |

     - 続けて `You can reply with the option letter ...` を表示する

   - 短答形式では:
     - 文脈に基づいた推奨回答を提示する
     - `**Suggested:** <answer> - <reasoning>` の形式にする
     - その後に短答フォーマット案内を出す

   - ユーザー回答後:
     - `yes`, `recommended`, `suggested` なら推薦 / 提案値を採用
     - それ以外は選択肢または 5 語以内制約へ適合するか確認
     - 曖昧なら同一質問内で再確認する
     - 問題なければ作業メモリへ記録し、次の質問へ進む

   - 次の場合は質問を止める:
     - 重要な曖昧さが早期に解消した
     - ユーザーが `done`, `good`, `no more` など完了意思を示した
     - 5 問に達した

   - 先の質問候補を先出ししてはならない
   - 最初から有効な質問がなければ、重要な曖昧さはないと即時報告する

5. **Integration after EACH accepted answer**:
   - 読み込んだ spec とその生内容をインメモリで保持する
   - セッション最初の反映時に `## Clarifications` セクションがなければ追加する
   - その下に今日の日付で `### Session YYYY-MM-DD` を作る
   - 受理後すぐに `- Q: <question> -> A: <final answer>` を追記する
   - さらに、内容に応じて最も適切なセクションへ即座に反映する
     - Functional ambiguity -> Functional Requirements
     - User interaction / actor distinction -> User Stories / Actors
     - Data shape / entities -> Data Model
     - Non-functional constraint -> Non-Functional / Quality Attributes
     - Edge case / negative flow -> Edge Cases / Error Handling
     - Terminology conflict -> 用語を正規化
   - 以前の曖昧文が無効になる場合は重複追記ではなく置換する
   - 各反映後に spec を保存してコンテキスト喪失を防ぐ
   - 見出し階層は保ち、無関係な並び替えはしない
   - 追加文は最小限かつ検証可能にする

6. **Validation**（各書き込み後と最終時）
   - Clarifications セッションには受理回答ごとにちょうど 1 行ある
   - 受理質問数は 5 以下
   - 反映後のセクションに、解消対象だった曖昧語が残っていない
   - 無効化された古い記述が残っていない
   - Markdown 構造が妥当で、新規見出しは `## Clarifications` と `### Session YYYY-MM-DD` のみ
   - 用語の一貫性が保たれている

7. 更新済み spec を `FEATURE_SPEC` に書き戻す。

8. **Report completion**:
   - 質問数と回答数
   - 更新した spec のパス
   - 触ったセクション一覧
   - 各カテゴリを `Resolved / Deferred / Clear / Outstanding` で示すカバレッジ表
   - Outstanding / Deferred があれば、`/speckit.plan` に進むべきか、後で `/speckit.clarify` を再実行すべきか推奨
   - 推奨次コマンド

Behavior rules:

- 重要な曖昧さが見つからなければ `No critical ambiguities detected worth formal clarification.` と返し、次へ進むことを勧める
- spec がなければ `/speckit.specify` を先に実行するよう案内する
- 新規質問は 5 問を超えない
- 機能的明確さを妨げない限り、技術スタックの推測質問は避ける
- `stop`, `done`, `proceed` などの早期終了意思を尊重する
- 質問不要ならコンパクトな全 Clear サマリを出して先へ促す
- 上限到達時に高影響未解決が残れば、理由付きで Deferred へ明示する

Context for prioritization: $ARGUMENTS
