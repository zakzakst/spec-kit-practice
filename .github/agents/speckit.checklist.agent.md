---
description: ユーザー要件に基づき、現在の機能向けカスタムチェックリストを生成する。
---

## チェックリストの目的：「自然言語に対するユニットテスト」 (Checklist Purpose: "Unit Tests for English")

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

**比喩**: spec が自然言語で書かれたコードなら、チェックリストはそのユニットテスト群である。実装が動くかではなく、要件が十分に書けているか、曖昧でないか、実装準備ができているかを検査する。

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## 実行手順 (Execution Steps)

1. **セットアップ (Setup)**: リポジトリルートで `.specify/scripts/powershell/check-prerequisites.ps1 -Json` を実行し、JSON から `FEATURE_DIR` と `AVAILABLE_DOCS` を取得する。
   - すべてのパスは絶対パスで扱うこと。
   - `I'm Groot` のようなシングルクォートを含む引数は `'I'\''m Groot'` のようにエスケープすること（可能なら `"I'm Groot"` を優先）。

2. **意図の明確化 (動的) (Clarify intent - dynamic)**: 事前に用意した質問集は使わず、最大 3 件までの初期確認質問を文脈に応じて生成する。各質問は次を満たすこと。
   - ユーザーの言い回しと spec / plan / tasks から抽出したシグナルに基づいて生成する
   - 回答次第でチェックリスト内容が実質的に変わるものだけを聞く
   - `$ARGUMENTS` ですでに明確なら個別にスキップする
   - 広さより精度を優先する

   生成アルゴリズム:
   1. シグナル抽出: ドメイン語（認証 (auth)、遅延 (latency)、ユーザー体験 (UX)、API など）、リスク指標（クリティカル (critical)、必須 (must)、コンプライアンス (compliance)）、ステークホルダーの手がかり（QA、レビュー (review)、セキュリティチーム (security team)）、成果物指定（アクセシビリティ (a11y)、ロールバック (rollback)、コントラクト (contracts)）
   2. シグナルを候補フォーカス領域へ最大 4 件までクラスタリングし、関連度順に並べる
   3. 明示されていなければ、想定利用者と利用タイミング（作成者、レビュアー、QA、リリース前）を推定する
   4. 欠けている軸を検出する: スコープ幅、深さ / 厳密さ、リスク重視、除外境界、測定可能な受け入れ基準
   5. 次の原型から質問を組み立てる:
      - スコープの洗練 (Scope refinement)
      - リスクの優先順位付け (Risk prioritization)
      - 深さの調整 (Depth calibration)
      - 対象者のフレーミング (Audience framing)
      - 境界の除外 (Boundary exclusion)
      - シナリオクラスのギャップ (Scenario class gap)

   質問フォーマット規則:
   - 選択肢を出す場合、`選択肢 (Option) | 候補 (Candidate) | なぜ重要か (Why It Matters)` のコンパクトな表を使う
   - 選択肢は最大 A-C とし、自由記述が適切なら表は省略する
   - ユーザーが既に言ったことを言い直させない
   - 根拠のない分類を作らない。不確かな場合は `Confirm whether X belongs in scope. (Xがスコープに含まれるか確認)` のように明示確認する

   対話できない場合のデフォルト:
   - 深さ (Depth): 標準 (Standard)
   - 対象者 (Audience): コード関連なら レビュアー (Reviewer) (PR)、それ以外は 作成者 (Author)
   - フォーカス (Focus): 関連度上位 2 クラスタ

   質問は `Q1` / `Q2` / `Q3` として出力する。回答後も Alternate / Exception / Recovery / Non-Functional のうち 2 種以上が不明な場合のみ、1 行理由付きで最大 2 問の追加質問 (`Q4` / `Q5`) を行ってよい。合計 5 問を超えてはならない。ユーザーが追加質問を断ったら打ち切る。

3. **ユーザーリクエストの理解 (Understand user request)**: `$ARGUMENTS` と確認回答を統合する。
   - チェックリストのテーマを導出する（例: security、review、deploy、ux）
   - ユーザーが明示した必須観点をまとめる
   - フォーカス選択をカテゴリ骨組みに反映する
   - spec / plan / tasks から不足文脈を補う（幻覚は禁止）

4. **機能コンテキストの読み込み (Load feature context)**: `FEATURE_DIR` から次を読む。
   - `spec.md`: 機能要件とスコープ
   - `plan.md`（あれば）: 技術詳細と依存
   - `tasks.md`（あれば）: 実装タスク

   **コンテキスト読み込み戦略 (Context Loading Strategy)**
   - アクティブな焦点領域に関係する部分だけを読む
   - 長文は生テキストではなく短いシナリオ / 要件箇条書きへ要約する
   - 不足が見つかったときだけ追加入手する
   - ソースが大きい場合は、中間要約を使い生文の貼り付けを避ける

5. **チェックリストの生成 (Generate checklist)**: 「実装」ではなく「要件」を検証するチェックリストを作る。
   - `FEATURE_DIR/checklists/` がなければ作成する
   - 短く説明的なファイル名を使う（例: `ux.md`, `api.md`, `security.md`）
   - 形式は `[domain].md`
   - 既存ファイルがある場合はそこへ追記する
   - 項目 ID は `CHK001` から連番
   - `/speckit.checklist` 実行ごとに**新しいファイルを作る**（既存チェックリストの上書きはしない）

   **基本原則 - 実装ではなく要件をテストする**
   各項目は必ず、要件そのものについて次を評価すること。
   - **完全性 (Completeness)**: 必要要件がそろっているか
   - **明確性 (Clarity)**: 曖昧さなく具体的か
   - **一貫性 (Consistency)**: 相互に矛盾していないか
   - **測定可能性 (Measurability)**: 客観的に検証可能か
   - **カバレッジ (Coverage)**: すべてのシナリオ / 例外を扱っているか

   **カテゴリ構造 (Category Structure)**
   - 要件の完全性 (Requirement Completeness)
   - 要件の明確性 (Requirement Clarity)
   - 要件の一貫性 (Requirement Consistency)
   - 受け入れ基準の品質 (Acceptance Criteria Quality)
   - シナリオのカバレッジ (Scenario Coverage)
   - エッジケースのカバレッジ (Edge Case Coverage)
   - 非機能要件 (Non-Functional Requirements)
   - 依存関係と前提条件 (Dependencies & Assumptions)
   - 曖昧さと衝突 (Ambiguities & Conflicts)

   **チェックリスト項目の書き方 — 「自然言語に対するユニットテスト」 (HOW TO WRITE CHECKLIST ITEMS - "Unit Tests for English")**

   誤（実装をテストしている）:
   - `Verify landing page displays 3 episode cards`（ランディングページに3つのエピソードカードが表示されることを確認する）
   - `Test hover states work on desktop`（デスクトップでホバー状態が機能することを確認する）
   - `Confirm logo click navigates home`（ロゴクリックでホームに遷移することを確認する）

   正（要件品質をテストしている）:
   - `Are the exact number and layout of featured episodes specified? [Completeness]`（フィーチャーされるエピソードの正確な数とレイアウトが指定されているか？ [完全性]）
   - `Is 'prominent display' quantified with specific sizing/positioning? [Clarity]`（「目立つ表示」は具体的なサイズ/位置で定量化されているか？ [明確性]）
   - `Are hover state requirements consistent across all interactive elements? [Consistency]`（ホバー状態の要件はすべてのインタラクティブ要素で一貫しているか？ [一貫性]）
   - `Are keyboard navigation requirements defined for all interactive UI? [Coverage]`（すべてのインタラクティブUIに対してキーボードナビゲーション要件が定義されているか？ [カバレッジ]）
   - `Is the fallback behavior specified when logo image fails to load? [Edge Cases]`（ロゴ画像の読み込み失敗時のフォールバック挙動が定義されているか？ [エッジケース]）
   - `Are loading states defined for asynchronous episode data? [Completeness]`（非同期のエピソードデータに対してローディング状態が定義されているか？ [完全性]）
   - `Does the spec define visual hierarchy for competing UI elements? [Clarity]`（仕様書は競合するUI要素の視覚的階層を定義しているか？ [明確性]）

   **項目構造 (ITEM STRUCTURE)**
   - 要件品質を問う質問形式にする
   - spec / plan に「書かれていること / いないこと」に焦点を当てる
   - 品質観点を角括弧で付ける `[Completeness]` など
   - 既存要件を確認する場合は `[Spec §X.Y]` を添える
   - 欠落確認には `[Gap]` を使う

   **品質次元ごとの例 (EXAMPLES BY QUALITY DIMENSION)**

   完全性 (Completeness):
   - `Are error handling requirements defined for all API failure modes? [Gap]`（すべてのAPI失敗モードに対してエラーハンドリング要件が定義されているか？ [ギャップ]）
   - `Are accessibility requirements specified for all interactive elements? [Completeness]`（すべてのインタラクティブ要素に対してアクセシビリティ要件が指定されているか？ [完全性]）
   - `Are mobile breakpoint requirements defined for responsive layouts? [Gap]`（レスポンシブレイアウトに対してモバイルのブレークポイント要件が定義されているか？ [ギャップ]）

   明確性 (Clarity):
   - `Is 'fast loading' quantified with specific timing thresholds? [Clarity, Spec §NFR-2]`（「高速ロード」は具体的な時間しきい値で定量化されているか？ [明確性, 仕様書 §NFR-2]）
   - `Are 'related episodes' selection criteria explicitly defined? [Clarity, Spec §FR-5]`（「関連エピソード」の選択基準は明示的に定義されているか？ [明確性, 仕様書 §FR-5]）
   - `Is 'prominent' defined with measurable visual properties? [Ambiguity, Spec §FR-4]`（「目立つ」は測定可能な視覚的プロパティで定義されているか？ [曖昧さ, 仕様書 §FR-4]）

   一貫性 (Consistency):
   - `Do navigation requirements align across all pages? [Consistency, Spec §FR-10]`（すべてのページでナビゲーション要件が整合しているか？ [一貫性, 仕様書 §FR-10]）
   - `Are card component requirements consistent between landing and detail pages? [Consistency]`（カードコンポーネントの要件はランディングページと詳細ページの間で一貫しているか？ [一貫性]）

   カバレッジ (Coverage):
   - `Are requirements defined for zero-state scenarios (no episodes)? [Coverage, Edge Case]`（データがゼロの状態（エピソードなし）のシナリオに対して要件が定義されているか？ [カバレッジ, エッジケース]）
   - `Are concurrent user interaction scenarios addressed? [Coverage, Gap]`（同時ユーザーインタラクションのシナリオは考慮されているか？ [カバレッジ, ギャップ]）
   - `Are requirements specified for partial data loading failures? [Coverage, Exception Flow]`（部分的なデータ読み込み失敗に対して要件が指定されているか？ [カバレッジ, 異常系フロー]）

   測定可能性 (Measurability):
   - `Are visual hierarchy requirements measurable/testable? [Acceptance Criteria, Spec §FR-1]`（視覚的階層の要件は測定可能/テスト可能か？ [受け入れ基準, 仕様書 §FR-1]）
   - `Can 'balanced visual weight' be objectively verified? [Measurability, Spec §FR-2]`（「バランスの取れた視覚的ウェイト」を客観的に検証できるか？ [測定可能性, 仕様書 §FR-2]）

   **シナリオの分類とカバレッジ (Scenario Classification & Coverage)**
   - 正常系 (Primary) / 代替系 (Alternate) / 異常系 (Exception) / 回復系 (Recovery) / 非機能 (Non-Functional) の各シナリオ群について要件の有無を確認する
   - 各群ごとに「完全・明確・一貫しているか」を問う
   - 欠落している場合は `intentionally excluded or missing? [Gap]`（意図的に除外されているのか、それとも欠落しているのか？ [ギャップ]）のように問う
   - 状態変更を伴う場合はロールバック / レジリエンスも対象に含める

   **トレーサビリティ要件 (Traceability Requirements)**
   - 最低でも 80% の項目に 1 つ以上のトレーサビリティ参照を含める
   - `[Spec §X.Y]` または `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]` を使う
   - ID 体系がなければ `Is a requirement & acceptance criteria ID scheme established? [Traceability]`（要件と受け入れ基準のID体系が確立されているか？ [トレーサビリティ]）を入れる

   **問題の顕在化と解決 (Surface & Resolve Issues)**
   - 曖昧さ (Ambiguities): `Is the term 'fast' quantified with specific metrics? [Ambiguity, Spec §NFR-1]`（「高速」という用語は具体的なメトリクスで定量化されているか？ [曖昧さ, 仕様書 §NFR-1]）
   - 衝突 (Conflicts): `Do navigation requirements conflict between §FR-10 and §FR-10a? [Conflict]`（§FR-10 と §FR-10a の間でナビゲーション要件が衝突しているか？ [衝突]）
   - 前提 (Assumptions): `Is the assumption of 'always available podcast API' validated? [Assumption]`（「常に利用可能なポッドキャストAPI」という前提は検証されているか？ [前提条件]）
   - 依存関係 (Dependencies): `Are external podcast API requirements documented? [Dependency, Gap]`（外部のポッドキャストAPI要件はドキュメント化されているか？ [依存関係, ギャップ]）
   - 定義漏れ (Missing definitions): `Is 'visual hierarchy' defined with measurable criteria? [Gap]`（「視覚的階層」は測定可能な基準で定義されているか？ [ギャップ]）

   **コンテンツの集約 (Content Consolidation)**
   - 候補が 40 件を超える場合はリスク / 影響度で優先順位付けする
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

6. **構造の参照 (Structure Reference)**: `.specify/templates/checklist-template.md` のタイトル、メタ、カテゴリ見出し、ID 形式に従う。テンプレートが使えない場合は、H1 タイトル、purpose / created メタ行、`##` カテゴリ見出し、その下に `- [ ] CHK### <item>` の形式で作成する。

7. **レポート (Report)**: 作成したチェックリストのフルパス、項目数、実行ごとに新規ファイルが作られることを報告する。次も要約すること。
   - 選択されたフォーカス領域 (focus areas)
   - 深さレベル (Depth level)
   - 対象者 / タイミング (Actor / timing)
   - 取り込んだユーザー指定の must-have 項目

**Important**: `/speckit.checklist` の各実行は、既存同名ファイルがない限り短く分かりやすい名前でチェックリストを作る。これにより:
- `ux.md`, `test.md`, `security.md` のような複数種類を共存できる
- 一目で用途が分かるファイル名にできる
- `checklists/` 配下で見つけやすい

不要になった古いチェックリストは適宜整理してよい。

## チェックリストのタイプ例とサンプル項目 (Example Checklist Types & Sample Items)

**UX要件の品質 (UX Requirements Quality):** `ux.md`

サンプル:
- `Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]`（視覚的階層の要件は測定可能な基準で定義されているか？ [明確性, 仕様書 §FR-1]）
- `Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]`（UI要素の数と配置は明示的に指定されているか？ [完全性, 仕様書 §FR-1]）
- `Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]`（インタラクション状態の要件（ホバー、フォーカス、アクティブ）が一貫して定義されているか？ [一貫性]）
- `Are accessibility requirements specified for all interactive elements? [Coverage, Gap]`（すべてのインタラクティブ要素に対してアクセシビリティ要件が指定されているか？ [カバレッジ, ギャップ]）
- `Is fallback behavior defined when images fail to load? [Edge Case, Gap]`（画像の読み込み失敗時のフォールバック挙動が定義されているか？ [エッジケース, ギャップ]）
- `Can 'prominent display' be objectively measured? [Measurability, Spec §FR-4]`（「目立つ表示」を客観的に測定できるか？ [測定可能性, 仕様書 §FR-4]）

**API要件の品質 (API Requirements Quality):** `api.md`

サンプル:
- `Are error response formats specified for all failure scenarios? [Completeness]`（すべての失敗シナリオに対してエラーレスポンス形式が指定されているか？ [完全性]）
- `Are rate limiting requirements quantified with specific thresholds? [Clarity]`（レート制限要件は具体的なしきい値で定量化されているか？ [明確性]）
- `Are authentication requirements consistent across all endpoints? [Consistency]`（認証要件はすべてのエンドポイントで一貫しているか？ [一貫性]）
- `Are retry/timeout requirements defined for external dependencies? [Coverage, Gap]`（外部依存関係に対してリトライ/タイムアウト要件が定義されているか？ [カバレッジ, ギャップ]）
- `Is versioning strategy documented in requirements? [Gap]`（バージョニング戦略は要件に文書化されているか？ [ギャップ]）

**パフォーマンス要件の品質 (Performance Requirements Quality):** `performance.md`

サンプル:
- `Are performance requirements quantified with specific metrics? [Clarity]`（パフォーマンス要件は具体的なメトリクスで定量化されているか？ [明確性]）
- `Are performance targets defined for all critical user journeys? [Coverage]`（すべての重要なユーザージャーニーに対してパフォーマンス目標が定義されているか？ [カバレッジ]）
- `Are performance requirements under different load conditions specified? [Completeness]`（異なる負荷条件下でのパフォーマンス要件が指定されているか？ [完全性]）
- `Can performance requirements be objectively measured? [Measurability]`（パフォーマンス要件を客観的に測定できるか？ [測定可能性]）
- `Are degradation requirements defined for high-load scenarios? [Edge Case, Gap]`（高負荷シナリオに対して機能低下（縮退運転）要件が定義されているか？ [エッジケース, ギャップ]）

**セキュリティ要件の品質 (Security Requirements Quality):** `security.md`

サンプル:
- `Are authentication requirements specified for all protected resources? [Coverage]`（保護されたすべてのリソースに対して認証要件が指定されているか？ [カバレッジ]）
- `Are data protection requirements defined for sensitive information? [Completeness]`（機密情報に対してデータ保護要件が定義されているか？ [完全性]）
- `Is the threat model documented and requirements aligned to it? [Traceability]`（脅威モデルが文書化され、要件がそれに沿っているか？ [トレーサビリティ]）
- `Are security requirements consistent with compliance obligations? [Consistency]`（セキュリティ要件はコンプライアンス義務と一貫しているか？ [一貫性]）
- `Are security failure/breach response requirements defined? [Gap, Exception Flow]`（セキュリティ侵害/失敗時の対応要件が定義されているか？ [ギャップ, 異常系フロー]）

## アンチパターン：やってはいけないこと (Anti-Examples: What NOT To Do)

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
