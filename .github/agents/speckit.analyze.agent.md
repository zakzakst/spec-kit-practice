---
description: タスク生成後に spec.md、plan.md、tasks.md 間の整合性と品質を、非破壊で横断分析する。
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理を進める前にユーザー入力を**必ず**考慮すること。

## Goal

実装に入る前に、3 つの主要成果物 (`spec.md`、`plan.md`、`tasks.md`) 間の不整合、重複、曖昧さ、記述不足を特定する。このコマンドは `/speckit.tasks` により完全な `tasks.md` が正常生成された後にのみ実行すること。

## Operating Constraints

**厳格な読み取り専用**: いかなるファイルも変更してはならない。構造化された分析レポートを出力すること。必要であれば改善案を提示してよいが、その後に編集系コマンドを使う場合はユーザーの明示的な承認が必要。

**Constitution Authority**: プロジェクト憲章 (`.specify/memory/constitution.md`) はこの分析の範囲では**交渉不可**。憲章との衝突は自動的に CRITICAL とし、原則を薄めたり再解釈したり黙殺したりせず、spec / plan / tasks 側を調整する必要がある。原則そのものを変更したい場合は、`/speckit.analyze` の外で別途明示的に憲章更新を行うこと。

## Execution Steps

### 1. Initialize Analysis Context

リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` を一度だけ実行し、JSON から `FEATURE_DIR` と `AVAILABLE_DOCS` を解析する。絶対パスは次のように導出する:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

必要ファイルが欠けていればエラーで中断し、足りない前提コマンドを実行するよう案内すること。
`I'm Groot` のように引数へシングルクォートが含まれる場合は、`'I'\''m Groot'` のようにエスケープすること（可能なら `"I'm Groot"` を優先）。

### 2. Load Artifacts (Progressive Disclosure)

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

### 3. Build Semantic Models

内部表現を作成する（生の成果物全文は出力しない）。

- **Requirements inventory**: すべての機能要件・非機能要件に安定キーを付与する。命令形の句から slug を導出すること（例: `user-can-upload-file`）
- **User story / action inventory**: ユーザー行動と受け入れ条件を離散的に整理する
- **Task coverage mapping**: 各タスクを 1 つ以上の要件またはストーリーに対応付ける。キーワードや ID、キーフレーズから推論してよい
- **Constitution rule set**: 原則名と MUST / SHOULD の規範文を抽出する

### 4. Detection Passes (Token-Efficient Analysis)

高シグナルな発見に集中すること。指摘は最大 50 件までとし、残りは要約へ集約する。

#### A. Duplication Detection

- ほぼ重複している要件を見つける
- 統合すべき低品質な言い換えを示す

#### B. Ambiguity Detection

- `fast`、`scalable`、`secure`、`intuitive`、`robust` のような測定基準のない曖昧な形容を検出する
- `TODO`、`TKTK`、`???`、`<placeholder>` など未解決プレースホルダーを検出する

#### C. Underspecification

- 動詞はあるが対象や測定可能な結果が欠ける要件
- 受け入れ条件との整合が弱いユーザーストーリー
- spec / plan に定義のないファイルやコンポーネントを参照しているタスク

#### D. Constitution Alignment

- MUST 原則に反する要件または plan 要素
- 憲章が要求する必須セクションや品質ゲートの欠落

#### E. Coverage Gaps

- 紐づくタスクが 0 件の要件
- どの要件 / ストーリーにも紐づかないタスク
- tasks に反映されていない非機能要件（性能、セキュリティなど）

#### F. Inconsistency

- 用語の揺れ
- plan にだけあるデータエンティティ、またはその逆
- タスク順序の矛盾（依存説明なしに基盤作業より先に統合作業がある等）
- 要件間の衝突（例: 一方は Next.js、他方は Vue を要求）

### 5. Severity Assignment

優先度は次の基準で判定する。

- **CRITICAL**: 憲章 MUST 違反、主要成果物の欠落、またはベースライン機能を阻害する無カバレッジ要件
- **HIGH**: 重複要件や矛盾要件、曖昧なセキュリティ / 性能属性、検証不能な受け入れ条件
- **MEDIUM**: 用語揺れ、非機能要件タスクの不足、境界ケースの記述不足
- **LOW**: 文言改善や、実行順序に影響しない軽微な冗長性

### 6. Produce Compact Analysis Report

次の構造で Markdown レポートを出力すること（ファイルは書き込まない）。

## Specification Analysis Report

| ID | Category | Severity | Location(s) | Summary | Recommendation |
|----|----------|----------|-------------|---------|----------------|
| A1 | Duplication | HIGH | spec.md:L120-134 | 似た要件が 2 件ある | より明確な記述へ統合する |

カテゴリ先頭文字を使った安定 ID を付け、1 件につき 1 行にまとめること。

**Coverage Summary Table**

| Requirement Key | Has Task? | Task IDs | Notes |
|-----------------|-----------|----------|-------|

**Constitution Alignment Issues:** （あれば）

**Unmapped Tasks:** （あれば）

**Metrics**

- Total Requirements
- Total Tasks
- Coverage %（1 件以上のタスクがある要件割合）
- Ambiguity Count
- Duplication Count
- Critical Issues Count

### 7. Provide Next Actions

レポート末尾に簡潔な Next Actions ブロックを付ける。

- CRITICAL がある場合: `/speckit.implement` 前に解消を推奨
- LOW / MEDIUM のみなら: 進行可能としつつ改善案を添える
- 明示的なコマンド候補を出す
  - 例: `Run /speckit.specify with refinement`
  - 例: `Run /speckit.plan to adjust architecture`
  - 例: `Manually edit tasks.md to add coverage for 'performance-metrics'`

### 8. Offer Remediation

最後に次のように尋ねること: `Would you like me to suggest concrete remediation edits for the top N issues?`

自動適用してはならない。

## Operating Principles

### Context Efficiency

- **Minimal high-signal tokens**: 網羅より実行可能な指摘を優先する
- **Progressive disclosure**: 成果物は段階的に読み、全文をだらだら出さない
- **Token-efficient output**: 表は 50 行まで、超過分は要約する
- **Deterministic results**: 変更がなければ再実行時も ID と集計が一貫する

### Analysis Guidelines

- **NEVER modify files**: これは読み取り専用分析である
- **NEVER hallucinate missing sections**: ないものは正確に「ない」と報告する
- **Prioritize constitution violations**: 憲章違反は常に最優先
- **Use examples over exhaustive rules**: 一般論より具体例を示す
- **Report zero issues gracefully**: 問題ゼロでも成功レポートとカバレッジ統計を出す

## Context

$ARGUMENTS
