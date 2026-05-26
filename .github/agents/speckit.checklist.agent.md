---
description: ユーザー要件に基づき、現在の機能向けカスタムチェックリストを生成する。
---

## Checklist Purpose: "Unit Tests for English"

**重要な考え方**: チェックリストは **要件文章に対するユニットテスト** である。特定ドメインにおける要件の品質、明確さ、完全性を検証するためのものだ。

**検証 / テスト実行のためのものではない**

- 誤: 「ボタンが正しくクリックできることを検証する」
- 誤: 「エラーハンドリングが動くことをテストする」
- 誤: 「API が 200 を返すことを確認する」
- 誤: 実装やコードが spec と一致しているかを確認すること

**要件品質の検証に使うもの**

- 正: 「すべてのカード種別について視覚的階層要件が定義されているか？」（完全性）
- 正: 「`prominent display` は具体的なサイズ / 位置で定量化されているか？」（明確性）
- 正: 「ホバー状態の要件は全インタラクティブ要素で一貫しているか？」（整合性）
- 正: 「キーボード操作向けアクセシビリティ要件が定義されているか？」（カバレッジ）
- 正: 「ロゴ画像の読み込み失敗時の挙動が spec に定義されているか？」（境界ケース）

**比喩**: spec が英語で書かれたコードなら、チェックリストはそのユニットテスト群である。実装が動くかではなく、要件が十分に書けているか、曖昧でないか、実装準備ができているかを検査する。

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## Execution Steps

1. **Setup**: リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json` を実行し、JSON から `FEATURE_DIR` と `AVAILABLE_DOCS` を取得する。
   - すべてのパスは絶対パスで扱うこと。
   - `I'm Groot` のようなシングルクォートを含む引数は `'I'\''m Groot'` のようにエスケープすること（可能なら `"I'm Groot"` を優先）。

2. **Clarify intent (dynamic)**: 事前に用意した質問集は使わず、最大 3 件までの初期確認質問を文脈に応じて生成する。各質問は次を満たすこと。
   - ユーザーの言い回しと spec / plan / tasks から抽出したシグナルに基づいて生成する
   - 回答次第でチェックリスト内容が実質的に変わるものだけを聞く
   - `$ARGUMENTS` ですでに明確なら個別にスキップする
   - 広さより精度を優先する

   生成アルゴリズム:
   1. シグナル抽出: ドメイン語（auth、latency、UX、API など）、リスク指標（`critical`、`must`、`compliance`）、ステークホルダー手掛かり（`QA`、`review`、`security team`）、成果物指定（`a11y`、`rollback`、`contracts`）
   2. シグナルを候補フォーカス領域へ最大 4 件までクラスタリングし、関連度順に並べる
   3. 明示されていなければ、想定利用者と利用タイミング（作者、レビュアー、QA、リリース前）を推定する
   4. 欠けている軸を検出する: スコープ幅、深さ / 厳密さ、リスク重視、除外境界、測定可能な受け入れ基準
   5. 次の原型から質問を組み立てる:
      - Scope refinement
      - Risk prioritization
      - Depth calibration
      - Audience framing
      - Boundary exclusion
      - Scenario class gap

   質問フォーマット規則:
   - 選択肢を出す場合、`Option | Candidate | Why It Matters` のコンパクトな表を使う
   - 選択肢は最大 A-C とし、自由記述が適切なら表は省略する
   - ユーザーが既に言ったことを言い直させない
   - 根拠のない分類を作らない。不確かな場合は `Confirm whether X belongs in scope.` のように明示確認する

   対話できない場合のデフォルト:
   - Depth: Standard
   - Audience: コード関連なら Reviewer (PR)、それ以外は Author
   - Focus: 関連度上位 2 クラスタ

   質問は `Q1` / `Q2` / `Q3` として出力する。回答後も Alternate / Exception / Recovery / Non-Functional のうち 2 種以上が不明な場合のみ、1 行理由付きで最大 2 問の追加質問 (`Q4` / `Q5`) を行ってよい。合計 5 問を超えてはならない。ユーザーが追加質問を断ったら打ち切る。

3. **Understand user request**: `$ARGUMENTS` と確認回答を統合する。
   - チェックリストのテーマを導出する（例: security、review、deploy、ux）
   - ユーザーが明示した必須観点をまとめる
   - フォーカス選択をカテゴリ骨組みに反映する
   - spec / plan / tasks から不足文脈を補う（幻覚は禁止）

4. **Load feature context**: `FEATURE_DIR` から次を読む。
   - `spec.md`: 機能要件とスコープ
   - `plan.md`（あれば）: 技術詳細と依存
   - `tasks.md`（あれば）: 実装タスク

   **Context Loading Strategy**
   - アクティブな焦点領域に関係する部分だけを読む
   - 長文は生テキストではなく短いシナリオ / 要件箇条書きへ要約する
   - 不足が見つかったときだけ追加入手する
   - ソースが大きい場合は、中間要約を使い生文の貼り付けを避ける

5. **Generate checklist**: 「実装」ではなく「要件」を検証するチェックリストを作る。
   - `FEATURE_DIR/checklists/` がなければ作成する
   - 短く説明的なファイル名を使う（例: `ux.md`, `api.md`, `security.md`）
   - 形式は `[domain].md`
   - 既存ファイルがある場合はそこへ追記する
   - 項目 ID は `CHK001` から連番
   - `/speckit.checklist` 実行ごとに**新しいファイルを作る**（既存チェックリストの上書きはしない）

   **基本原則 - 実装ではなく要件をテストする**
   各項目は必ず、要件そのものについて次を評価すること。
   - **Completeness**: 必要要件がそろっているか
   - **Clarity**: 曖昧さなく具体的か
   - **Consistency**: 相互に矛盾していないか
   - **Measurability**: 客観的に検証可能か
   - **Coverage**: すべてのシナリオ / 例外を扱っているか

   **Category Structure**
   - Requirement Completeness
   - Requirement Clarity
   - Requirement Consistency
   - Acceptance Criteria Quality
   - Scenario Coverage
   - Edge Case Coverage
   - Non-Functional Requirements
   - Dependencies & Assumptions
   - Ambiguities & Conflicts

   **HOW TO WRITE CHECKLIST ITEMS - "Unit Tests for English"**

   誤（実装をテストしている）:
   - `Verify landing page displays 3 episode cards`
   - `Test hover states work on desktop`
   - `Confirm logo click navigates home`

   正（要件品質をテストしている）:
   - `Are the exact number and layout of featured episodes specified? [Completeness]`
   - `Is 'prominent display' quantified with specific sizing/positioning? [Clarity]`
   - `Are hover state requirements consistent across all interactive elements? [Consistency]`
   - `Are keyboard navigation requirements defined for all interactive UI? [Coverage]`
   - `Is the fallback behavior specified when logo image fails to load? [Edge Cases]`
   - `Are loading states defined for asynchronous episode data? [Completeness]`
   - `Does the spec define visual hierarchy for competing UI elements? [Clarity]`

   **ITEM STRUCTURE**
   - 要件品質を問う質問形式にする
   - spec / plan に「書かれていること / いないこと」に焦点を当てる
   - 品質観点を角括弧で付ける `[Completeness]` など
   - 既存要件を確認する場合は `[Spec §X.Y]` を添える
   - 欠落確認には `[Gap]` を使う

   **EXAMPLES BY QUALITY DIMENSION**

   Completeness:
   - `Are error handling requirements defined for all API failure modes? [Gap]`
   - `Are accessibility requirements specified for all interactive elements? [Completeness]`
   - `Are mobile breakpoint requirements defined for responsive layouts? [Gap]`

   Clarity:
   - `Is 'fast loading' quantified with specific timing thresholds? [Clarity, Spec §NFR-2]`
   - `Are 'related episodes' selection criteria explicitly defined? [Clarity, Spec §FR-5]`
   - `Is 'prominent' defined with measurable visual properties? [Ambiguity, Spec §FR-4]`

   Consistency:
   - `Do navigation requirements align across all pages? [Consistency, Spec §FR-10]`
   - `Are card component requirements consistent between landing and detail pages? [Consistency]`

   Coverage:
   - `Are requirements defined for zero-state scenarios (no episodes)? [Coverage, Edge Case]`
   - `Are concurrent user interaction scenarios addressed? [Coverage, Gap]`
   - `Are requirements specified for partial data loading failures? [Coverage, Exception Flow]`

   Measurability:
   - `Are visual hierarchy requirements measurable/testable? [Acceptance Criteria, Spec §FR-1]`
   - `Can 'balanced visual weight' be objectively verified? [Measurability, Spec §FR-2]`

   **Scenario Classification & Coverage**
   - Primary / Alternate / Exception / Recovery / Non-Functional の各シナリオ群について要件有無を確認する
   - 各群ごとに「完全・明確・一貫しているか」を問う
   - 欠落している場合は `intentionally excluded or missing? [Gap]` のように問う
   - 状態変更を伴う場合はロールバック / レジリエンスも対象に含める

   **Traceability Requirements**
   - 最低でも 80% の項目に 1 つ以上のトレーサビリティ参照を含める
   - `[Spec §X.Y]` または `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]` を使う
   - ID 体系がなければ `Is a requirement & acceptance criteria ID scheme established? [Traceability]` を入れる

   **Surface & Resolve Issues**
   - Ambiguities: `Is the term 'fast' quantified with specific metrics? [Ambiguity, Spec §NFR-1]`
   - Conflicts: `Do navigation requirements conflict between §FR-10 and §FR-10a? [Conflict]`
   - Assumptions: `Is the assumption of 'always available podcast API' validated? [Assumption]`
   - Dependencies: `Are external podcast API requirements documented? [Dependency, Gap]`
   - Missing definitions: `Is 'visual hierarchy' defined with measurable criteria? [Gap]`

   **Content Consolidation**
   - 候補が 40 件を超える場合はリスク / 影響で優先づけする
   - 同じ観点を検査する重複項目は統合する
   - 低影響な境界ケースが多い場合は 1 項目へまとめてもよい

   **絶対に禁止**
   - `Verify` / `Test` / `Confirm` / `Check` で始まり、実装挙動を検証する項目
   - コード実行、ユーザー操作、システム動作への直接言及
   - `displays correctly`, `works properly`, `functions as expected`
   - `click`, `navigate`, `render`, `load`, `execute`
   - テストケース、テスト計画、QA 手順
   - フレームワークや API、アルゴリズム等の実装詳細

   **推奨パターン**
   - `Are [requirement type] defined/specified/documented for [scenario]?`
   - `Is [vague term] quantified/clarified with specific criteria?`
   - `Are requirements consistent between [section A] and [section B]?`
   - `Can [requirement] be objectively measured/verified?`
   - `Are [edge cases/scenarios] addressed in requirements?`
   - `Does the spec define [missing aspect]?`

6. **Structure Reference**: `.specify/templates/checklist-template.md` のタイトル、メタ、カテゴリ見出し、ID 形式に従う。テンプレートが使えない場合は、H1 タイトル、purpose / created メタ行、`##` カテゴリ見出し、その下に `- [ ] CHK### <item>` の形式で作成する。

7. **Report**: 作成したチェックリストのフルパス、項目数、実行ごとに新規ファイルが作られることを報告する。次も要約すること。
   - 選択された focus areas
   - Depth level
   - Actor / timing
   - 取り込んだユーザー指定 must-have 項目

**Important**: `/speckit.checklist` の各実行は、既存同名ファイルがない限り短く分かりやすい名前でチェックリストを作る。これにより:

- `ux.md`, `test.md`, `security.md` のような複数種類を共存できる
- 一目で用途が分かるファイル名にできる
- `checklists/` 配下で見つけやすい

不要になった古いチェックリストは適宜整理してよい。

## Example Checklist Types & Sample Items

**UX Requirements Quality:** `ux.md`

サンプル:

- `Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]`
- `Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]`
- `Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]`
- `Are accessibility requirements specified for all interactive elements? [Coverage, Gap]`
- `Is fallback behavior defined when images fail to load? [Edge Case, Gap]`
- `Can 'prominent display' be objectively measured? [Measurability, Spec §FR-4]`

**API Requirements Quality:** `api.md`

サンプル:

- `Are error response formats specified for all failure scenarios? [Completeness]`
- `Are rate limiting requirements quantified with specific thresholds? [Clarity]`
- `Are authentication requirements consistent across all endpoints? [Consistency]`
- `Are retry/timeout requirements defined for external dependencies? [Coverage, Gap]`
- `Is versioning strategy documented in requirements? [Gap]`

**Performance Requirements Quality:** `performance.md`

サンプル:

- `Are performance requirements quantified with specific metrics? [Clarity]`
- `Are performance targets defined for all critical user journeys? [Coverage]`
- `Are performance requirements under different load conditions specified? [Completeness]`
- `Can performance requirements be objectively measured? [Measurability]`
- `Are degradation requirements defined for high-load scenarios? [Edge Case, Gap]`

**Security Requirements Quality:** `security.md`

サンプル:

- `Are authentication requirements specified for all protected resources? [Coverage]`
- `Are data protection requirements defined for sensitive information? [Completeness]`
- `Is the threat model documented and requirements aligned to it? [Traceability]`
- `Are security requirements consistent with compliance obligations? [Consistency]`
- `Are security failure/breach response requirements defined? [Gap, Exception Flow]`

## Anti-Examples: What NOT To Do

**誤 - 実装をテストしている例**

```markdown
- [ ] CHK001 - Verify landing page displays 3 episode cards [Spec §FR-001]
- [ ] CHK002 - Test hover states work correctly on desktop [Spec §FR-003]
- [ ] CHK003 - Confirm logo click navigates to home page [Spec §FR-010]
- [ ] CHK004 - Check that related episodes section shows 3-5 items [Spec §FR-005]
```

**正 - 要件品質をテストしている例**

```markdown
- [ ] CHK001 - Are the number and layout of featured episodes explicitly specified? [Completeness, Spec §FR-001]
- [ ] CHK002 - Are hover state requirements consistently defined for all interactive elements? [Consistency, Spec §FR-003]
- [ ] CHK003 - Are navigation requirements clear for all clickable brand elements? [Clarity, Spec §FR-010]
- [ ] CHK004 - Is the selection criteria for related episodes documented? [Gap, Spec §FR-005]
- [ ] CHK005 - Are loading state requirements defined for asynchronous episode data? [Gap]
- [ ] CHK006 - Can "visual hierarchy" requirements be objectively measured? [Measurability, Spec §FR-001]
```

**違いの要点**

- 誤: システムが正しく動くかを試している
- 正: 要件が正しく書かれているかを試している
- 誤: 挙動の検証
- 正: 要件品質の検証
- 誤: `Does it do X?`
- 正: `Is X clearly specified?`
