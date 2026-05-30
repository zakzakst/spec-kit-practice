---
description: タスク生成後に spec.md、plan.md、tasks.md 間の整合性と品質を、非破壊で横断分析する。
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理を進める前にユーザー入力を**必ず**考慮すること。

## ゴール (Goal)

実装に入る前に、3 つの主要成果物 (`spec.md`、`plan.md`、`tasks.md`) 間の不整合、重複、曖昧さ、記述不足を特定する。このコマンドは `/speckit.tasks` により完全な `tasks.md` が正常生成された後にのみ実行すること。

## 動作制約 (Operating Constraints)

**厳格な読み取り専用**: いかなるファイルも変更してはならない。構造化された分析レポートを出力すること。必要であれば改善案を提示してよいが、その後に編集系コマンドを使う場合はユーザーの明示的な承認が必要。

**憲章の権限 (Constitution Authority)**: プロジェクト憲章 (`.specify/memory/constitution.md`) はこの分析の範囲では**交渉不可**。憲章との衝突は自動的に CRITICAL とし、原則を薄めたり再解釈したり黙殺したりせず、spec / plan / tasks 側を調整する必要がある。原則そのものを変更したい場合は、`/speckit.analyze` の外で別途明示的に憲章更新を行うこと。

## 実行手順 (Execution Steps)

### 1. 分析コンテキストの初期化 (Initialize Analysis Context)

リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` を一度だけ実行し、JSON から `FEATURE_DIR` と `AVAILABLE_DOCS` を解析する。絶対パスは次のように導出する:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

必要ファイルが欠けていればエラーで中断し、足りない前提コマンドを実行するよう案内すること。
`I'm Groot` のように引数へシングルクォートが含まれる場合は、`'I'\''m Groot'` のようにエスケープすること（可能なら `"I'm Groot"` を優先）。

### 2. 成果物の読み込み (段階的開示) (Load Artifacts - Progressive Disclosure)

各成果物から本当に必要な最小限の情報だけを読むこと。

**spec.md から読むもの**

- Overview / Context
- Functional Requirements
- Non-Functional Requirements
- User Stories
- Edge Cases（あれば）

**plan.md から読むもの**

- Architecture / stack choices
- Data Model への参照
- Phases
- Technical constraints

**tasks.md から読むもの**

- Task IDs
- Descriptions
- Phase grouping
- Parallel markers [P]
- Referenced file paths

**constitution から読むもの**

- `.specify/memory/constitution.md` を読み、原則検証に使う

### 3. セマンティックモデルの構築 (Build Semantic Models)

内部表現を作成する（生の成果物全文は出力しない）。

- **要件インベントリ (Requirements inventory)**: すべての機能要件・非機能要件に安定キーを付与する。命令形の句から slug を導出すること（例: `user-can-upload-file`）
- **ユーザー行動/アクションインベントリ (User story / action inventory)**: ユーザー行動と受け入れ条件を離散的に整理する
- **タスクカバレッジマッピング (Task coverage mapping)**: 各タスクを 1 つ以上の要件またはストーリーに対応付ける。キーワードや ID、キーフレーズから推論してよい
- **憲章ルールセット (Constitution rule set)**: 原則名と MUST / SHOULD の規範文を抽出する

### 4. 検出パス (トークン効率分析) (Detection Passes - Token-Efficient Analysis)

高シグナルな発見に集中すること。指摘は最大 50 件までとし、残りは要約へ集約する。

#### A. 重複検出 (Duplication Detection)

- ほぼ重複している要件を見つける
- 統合すべき低品質な言い換えを示す

#### B. 曖昧さ検出 (Ambiguity Detection)

- `fast`、`scalable`、`secure`、`intuitive`、`robust` のような測定基準のない曖昧な形容を検出する
- `TODO`、`TKTK`、`???`、`<placeholder>` など未解決プレースホルダーを検出する

#### C. 記述不足の検出 (Underspecification)

- 動詞はあるが対象や測定可能な結果が欠ける要件
- 受け入れ条件との整合が弱いユーザーストーリー
- spec / plan に定義のないファイルやコンポーネントを参照しているタスク

#### D. 憲章との整合性 (Constitution Alignment)

- MUST 原則に反する要件または plan 要素
- 憲章が要求する必須セクションや品質ゲートの欠落

#### E. カバレッジギャップ (Coverage Gaps)

- 紐づくタスクが 0 件の要件
- どの要件 / ストーリーにも紐づかないタスク
- tasks に反映されていない非機能要件（性能、セキュリティなど）

#### F. 不整合の検出 (Inconsistency)

- 用語の揺れ
- plan にだけあるデータエンティティ、またはその逆
- タスク順序の矛盾（依存説明なしに基盤作業より先に統合作業がある等）
- 要件間の衝突（例: 一方は Next.js、他方は Vue を要求）

### 5. 重要度の割り当て (Severity Assignment)

優先度は次の基準で判定する。

- **CRITICAL**: 憲章 MUST 違反、主要成果物の欠落、またはベースライン機能を阻害する無カバレッジ要件
- **HIGH**: 重複要件や矛盾要件、曖昧なセキュリティ / 性能属性、検証不能な受け入れ条件
- **MEDIUM**: 用語揺れ、非機能要件タスクの不足、境界ケースの記述不足
- **LOW**: 文言改善や、実行順序に影響しない軽微な冗長性

### 6. コンパクトな分析レポートの作成 (Produce Compact Analysis Report)

次の構造で Markdown レポートを出力すること（ファイルは書き込まない）。

## Specification Analysis Report

| ID | カテゴリ (Category) | 重要度 (Severity) | 場所 (Location(s)) | 要約 (Summary) | 推奨対策 (Recommendation) |
|----|----------|----------|-------------|---------|----------------|
| A1 | Duplication | HIGH | spec.md:L120-134 | 似た要件が 2 件ある | より明確な記述へ統合する |

カテゴリ先頭文字を使った安定 ID を付け、1 件につき 1 行にまとめること。

**カバレッジ要約表 (Coverage Summary Table)**

| 要件キー (Requirement Key) | タスクあり? (Has Task?) | タスクID (Task IDs) | メモ (Notes) |
|-----------------|-----------|----------|-------|

**憲章との不整合 (Constitution Alignment Issues):** （あれば）

**マッピングされていないタスク (Unmapped Tasks):** （あれば）

**メトリクス (Metrics)**

- 総要件数 (Total Requirements)
- 総タスク数 (Total Tasks)
- カバレッジ率 % (Coverage %):（1 件以上のタスクがある要件割合）
- 曖昧な箇所の数 (Ambiguity Count)
- 重複の数 (Duplication Count)
- 重大な問題の数 (Critical Issues Count)

### 7. 次のアクションの提示 (Provide Next Actions)

レポート末尾に簡潔な Next Actions ブロックを付ける。

- CRITICAL がある場合: `/speckit.implement` 前に解消を推奨
- LOW / MEDIUM のみなら: 進行可能としつつ改善案を添える
- 明示的なコマンド候補を出す
  - 例: `Run /speckit.specify with refinement`
  - 例: `Run /speckit.plan to adjust architecture`
  - 例: `Manually edit tasks.md to add coverage for 'performance-metrics'`

### 8. 修正案の提示 (Offer Remediation)

最後に次のように尋ねること: `Would you like me to suggest concrete remediation edits for the top N issues? (上位 N 件の問題に対して具体的な修正案を提示しましょうか？)`

自動適用してはならない。

## 動作原則 (Operating Principles)

### コンテキストの効率性 (Context Efficiency)

- **最小限の高シグナルトークン (Minimal high-signal tokens)**: 網羅より実行可能な指摘を優先する
- **段階的開示 (Progressive disclosure)**: 成果物は段階的に読み、全文をだらだら出さない
- **トークン効率の良い出力 (Token-efficient output)**: 表は 50 行まで、超過分は要約する
- **決定論的な結果 (Deterministic results)**: 変更がなければ再実行時も ID と集計が一貫する

### 分析ガイドライン (Analysis Guidelines)

- **ファイルの変更は絶対に禁止 (NEVER modify files)**: これは読み取り専用分析である
- **欠落したセクションの捏造は絶対に禁止 (NEVER hallucinate missing sections)**: ないものは正確に「ない」と報告する
- **憲章違反を最優先 (Prioritize constitution violations)**: 憲章違反は常に最優先
- **網羅的なルールより具体例を使用 (Use examples over exhaustive rules)**: 一般論より具体例を示す
- **問題ゼロでも正常レポートを出力 (Report zero issues gracefully)**: 問題ゼロでも成功レポートとカバレッジ統計を出す

## コンテキスト (Context)

$ARGUMENTS
