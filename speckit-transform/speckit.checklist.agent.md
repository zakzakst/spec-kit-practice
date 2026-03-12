---
description: ユーザーの要件に基づいて、現在の機能のカスタム チェックリストを生成します。
---

## チェックリストの目的: 「英語のユニットテスト」

**重要な概念**: チェックリストは**要件記述のためのユニットテスト**であり、特定のドメインにおける要件の品質、明確さ、完全性を検証します。

**検証/テスト用ではありません**:

- ❌ 「ボタンが正しくクリックされることを確認する」ではありません
- ❌ 「エラー処理が機能するかどうかをテストする」ではない
- ❌ 「API が 200 を返すことを確認する」ではありません
- ❌ コード/実装が仕様と一致しているかどうかを確認しない

**要件品質検証**:

- ✅ 「すべてのカードタイプに対して視覚的な階層要件が定義されていますか？」（完全性）
- ✅ 「『目立つ表示』は、具体的なサイズや位置で定量化されていますか？」（明確さ）
- ✅ 「ホバー状態の要件は、すべてのインタラクティブ要素にわたって一貫していますか？」（一貫性）
- ✅ 「キーボードナビゲーションのアクセシビリティ要件は定義されていますか?」（カバレッジ）
- ✅ 「ロゴ画像の読み込みに失敗した場合、何が起こるか仕様で定義されていますか？」（エッジケース）

**比喩**: 仕様書が英語で書かれたコードである場合、チェックリストはそのユニットテストスイートです。テストするのは、要件が適切に記述され、完全で、明確で、実装の準備ができているかどうかであり、実装が機能するかどうかではありません。

## ユーザー入力

```text
$ARGUMENTS
```

続行する前に、ユーザー入力を考慮する必要があります (空でない場合)。

## 実行手順

1. **設定**: リポジトリのルートから `.specify/scripts/powershell/check-prerequisites.ps1 -Json` を実行し、FEATURE_DIR と AVAILABLE_DOCS リストの JSON を解析します。
   - すべてのファイル パスは絶対パスである必要があります。
   - "I'm Groot" のような引数内の一重引用符の場合は、エスケープ構文を使用します。例: 'I'\''m Groot' (または可能であれば二重引用符を使用します: "I'm Groot")。

2. **意図を明確にする（動的）**: 文脈を明確にするための最初の質問を最大3つ作成する（事前に用意されたカタログは不要）:

- ユーザーのフレーズから生成され、仕様/計画/タスクから抽出されたシグナル
- チェックリストの内容を大幅に変更する情報についてのみ質問する
- `$ARGUMENTS` で既に明確な場合は個別にスキップされます
- 広さよりも精度を優先する

生成アルゴリズム:

1. シグナルを抽出します。特徴ドメインのキーワード (例: 認証、レイテンシ、UX、API)、リスク指標 (「クリティカル」、「必須」、「コンプライアンス」)、利害関係者のヒント (「QA」、「レビュー」、「セキュリティ チーム」)、および明示的な成果物 (「a11y」、「ロールバック」、「契約」) を抽出します。
2. 信号を、関連性に基づいてランク付けされた候補の焦点領域 (最大 4 つ) にクラスター化します。
3. 明確でない場合は、想定される対象者とタイミング (作成者、レビュー担当者、QA、リリース) を特定します。
4. 欠落している次元を検出します: 範囲の広さ、深さ/厳密さ、リスクの強調、除外の境界、測定可能な受け入れ基準。
5. これらのアーキタイプから選択した質問を策定する:
   - スコープの絞り込み (例: 「X および Y との統合タッチポイントを含めるか、ローカル モジュールの正確性に限定するか?」)
   - リスクの優先順位付け (例: 「これらの潜在的なリスク領域のうち、どの領域に必須のゲーティング チェックを実施する必要がありますか?」)
   - 深さの調整 (例: 「これは軽量のコミット前健全性リストですか、それとも正式なリリース ゲートですか?」)
   - 対象者のフレーミング (例: 「これは PR レビュー中に作成者のみが使用するのか、それとも同僚が使用するのか?」)
   - 境界除外 (例: 「このラウンドではパフォーマンス チューニング項目を明示的に除外する必要がありますか?」)
   - シナリオ クラスのギャップ (例: 「回復フローが検出されませんでした。ロールバック/部分的な障害パスは範囲内ですか?」)

質問のフォーマットルール:

- 選択肢を提示する場合は、選択肢 | 候補 | それが重要な理由という列を持つコンパクトな表を作成します。
- 選択肢はA～Eまでに制限する。自由形式の回答の方が明確な場合は表を省略する。
- ユーザーに既に言ったことをもう一度言うように求めない。
- 推測的なカテゴリー（幻覚など）は避けてください。不明な場合は、「Xが範囲内かどうか確認してください」と明確に質問してください。

対話が不可能な場合のデフォルト:

- 深さ: 標準
- 対象者: コード関連の場合はレビュー担当者 (PR)、それ以外の場合は著者
- 焦点: 上位2つの関連クラスター

質問を出力します（ラベルはQ1/Q2/Q3）。回答後、2つ以上のシナリオクラス（代替/例外/復旧/非機能領域）が不明瞭なままの場合、最大2つのターゲットフォローアップ（Q4/Q5）を、それぞれ1行の理由（例：「未解決の復旧パスリスク」）とともに尋ねることができます。質問の総数は5つを超えないようにしてください。ユーザーが明示的にそれ以上の質問を拒否した場合は、エスカレーションをスキップしてください。

3. **ユーザーのリクエストを理解する**: `$ARGUMENTS` + 明確な回答を組み合わせる:
   - チェックリストのテーマを導き出す（例：セキュリティ、レビュー、デプロイ、UX）
   - ユーザーが明示的に言及した必須アイテムを統合する
   - フォーカス選択をカテゴリスキャフォールディングにマッピングする
   - 仕様/計画/タスクから不足しているコンテキストを推測する（幻覚はしない）

4. **フィーチャコンテキストを読み込む**: FEATURE_DIRから読み取る:
   - spec.md: 機能要件と範囲
   - plan.md (if exists): 技術的な詳細、依存関係
   - tasks.md (if exists): 実装タスク

   **コンテキスト読み込み戦略**:
   - アクティブなフォーカス領域に関連する必要な部分のみをロードする（ファイル全体のダンプを避ける）
   - 長いセクションを簡潔なシナリオ/要件の箇条書きにまとめることを優先します
   - 段階的な開示を使用する: ギャップが検出された場合にのみ、後続の検索を追加します
   - ソースドキュメントが大きい場合は、生のテキストを埋め込む代わりに中間要約項目を生成します。

5. **チェックリストを生成する** - 「要件のユニットテスト」を作成する:
   - `FEATURE_DIR/checklists/` ディレクトリが存在しない場合は作成します
   - 一意のチェックリストファイル名を生成する:
     - ドメインに基づいた短くて説明的な名前を使用します（例：`ux.md`、`api.md`、`security.md`）
     - フォーマット: `[domain].md`
     - ファイルが存在する場合は、既存のファイルに追加します
   - CHK001から順に番号を付けます
   - `/speckit.checklist` を実行するたびに新しいファイルが作成されます (既存のチェックリストが上書きされることはありません)

**基本原則 - 実装ではなく要件をテストする**:
すべてのチェックリスト項目は、要件自体を評価する必要があります:

- **完全**: 必要な要件はすべて揃っていますか?
- **明瞭さ**: 要件は明確かつ具体的ですか?
- **一貫性**: 要件は互いに一致していますか?
- **測定可能性**: 要件は客観的に検証できますか?
- **カバレッジ**: すべてのシナリオ/エッジケースが対処されていますか?

**カテゴリー構造** - 要件品質ディメンション別にアイテムをグループ化:

- **要件の完全性** (必要な要件はすべて文書化されていますか?)
- **要件の明確化** (要件は具体的かつ明確ですか?)
- **要件の一貫性** (要件は矛盾なく一致していますか?)
- **受け入れ基準品質** (成功基準は測定可能ですか?)
- **シナリオカバレッジ** (すべてのフロー/ケースがカバーされていますか?)
- **エッジケースカバレッジ** (境界条件は定義されていますか?)
- **非機能要件** (パフォーマンス、セキュリティ、アクセシビリティなどが指定されていますか?)
- **依存関係と仮定** (それらは文書化され、検証されていますか?)
- **曖昧さと対立** (何を明確にする必要があるでしょうか？)

**チェックリスト項目の書き方 - 「英語のユニットテスト」**:

❌ **間違っている** （テスト実装）:

- 「ランディングページに 3 つのエピソード カードが表示されていることを確認します」
- 「デスクトップでホバー状態をテストする」
- 「ロゴをクリックするとホームへ移動します」

✅ **正しい** （テスト要件の品質）:

- 「特集エピソードの正確な数とレイアウトが指定されていますか？」[完全性]
- 「『目立つ表示』は、具体的なサイズや位置で定量化されていますか？」[明確さ]
- 「ホバー状態の要件は、すべてのインタラクティブ要素にわたって一貫していますか？」[一貫性]
- 「すべてのインタラクティブ UI に対してキーボード ナビゲーションの要件が定義されていますか?」[カバレッジ]
- 「ロゴ画像の読み込みに失敗した場合のフォールバック動作は指定されていますか？」[エッジケース]
- 「非同期エピソードデータの読み込み状態は定義されていますか？」[完全性]
- 「仕様では、競合する UI 要素の視覚的な階層が定義されていますか？」[Clarity]

**ITEM STRUCTURE**:
Each item should follow this pattern:

- Question format asking about requirement quality
- Focus on what's WRITTEN (or not written) in the spec/plan
- Include quality dimension in brackets [Completeness/Clarity/Consistency/etc.]
- Reference spec section `[Spec §X.Y]` when checking existing requirements
- Use `[Gap]` marker when checking for missing requirements

**EXAMPLES BY QUALITY DIMENSION**:

Completeness:

- "Are error handling requirements defined for all API failure modes? [Gap]"
- "Are accessibility requirements specified for all interactive elements? [Completeness]"
- "Are mobile breakpoint requirements defined for responsive layouts? [Gap]"

Clarity:

- "Is 'fast loading' quantified with specific timing thresholds? [Clarity, Spec §NFR-2]"
- "Are 'related episodes' selection criteria explicitly defined? [Clarity, Spec §FR-5]"
- "Is 'prominent' defined with measurable visual properties? [Ambiguity, Spec §FR-4]"

Consistency:

- "Do navigation requirements align across all pages? [Consistency, Spec §FR-10]"
- "Are card component requirements consistent between landing and detail pages? [Consistency]"

Coverage:

- "Are requirements defined for zero-state scenarios (no episodes)? [Coverage, Edge Case]"
- "Are concurrent user interaction scenarios addressed? [Coverage, Gap]"
- "Are requirements specified for partial data loading failures? [Coverage, Exception Flow]"

Measurability:

- "Are visual hierarchy requirements measurable/testable? [Acceptance Criteria, Spec §FR-1]"
- "Can 'balanced visual weight' be objectively verified? [Measurability, Spec §FR-2]"

**Scenario Classification & Coverage** (Requirements Quality Focus):

- Check if requirements exist for: Primary, Alternate, Exception/Error, Recovery, Non-Functional scenarios
- For each scenario class, ask: "Are [scenario type] requirements complete, clear, and consistent?"
- If scenario class missing: "Are [scenario type] requirements intentionally excluded or missing? [Gap]"
- Include resilience/rollback when state mutation occurs: "Are rollback requirements defined for migration failures? [Gap]"

**Traceability Requirements**:

- MINIMUM: ≥80% of items MUST include at least one traceability reference
- Each item should reference: spec section `[Spec §X.Y]`, or use markers: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`
- If no ID system exists: "Is a requirement & acceptance criteria ID scheme established? [Traceability]"

**Surface & Resolve Issues** (Requirements Quality Problems):
Ask questions about the requirements themselves:

- Ambiguities: "Is the term 'fast' quantified with specific metrics? [Ambiguity, Spec §NFR-1]"
- Conflicts: "Do navigation requirements conflict between §FR-10 and §FR-10a? [Conflict]"
- Assumptions: "Is the assumption of 'always available podcast API' validated? [Assumption]"
- Dependencies: "Are external podcast API requirements documented? [Dependency, Gap]"
- Missing definitions: "Is 'visual hierarchy' defined with measurable criteria? [Gap]"

**Content Consolidation**:

- Soft cap: If raw candidate items > 40, prioritize by risk/impact
- Merge near-duplicates checking the same requirement aspect
- If >5 low-impact edge cases, create one item: "Are edge cases X, Y, Z addressed in requirements? [Coverage]"

**🚫 ABSOLUTELY PROHIBITED** - These make it an implementation test, not a requirements test:

- ❌ Any item starting with "Verify", "Test", "Confirm", "Check" + implementation behavior
- ❌ References to code execution, user actions, system behavior
- ❌ "Displays correctly", "works properly", "functions as expected"
- ❌ "Click", "navigate", "render", "load", "execute"
- ❌ Test cases, test plans, QA procedures
- ❌ Implementation details (frameworks, APIs, algorithms)

**✅ REQUIRED PATTERNS** - These test requirements quality:

- ✅ "Are [requirement type] defined/specified/documented for [scenario]?"
- ✅ "Is [vague term] quantified/clarified with specific criteria?"
- ✅ "Are requirements consistent between [section A] and [section B]?"
- ✅ "Can [requirement] be objectively measured/verified?"
- ✅ "Are [edge cases/scenarios] addressed in requirements?"
- ✅ "Does the spec define [missing aspect]?"

6. **Structure Reference**: Generate the checklist following the canonical template in `.specify/templates/checklist-template.md` for title, meta section, category headings, and ID formatting. If template is unavailable, use: H1 title, purpose/created meta lines, `##` category sections containing `- [ ] CHK### <requirement item>` lines with globally incrementing IDs starting at CHK001.

7. **Report**: Output full path to created checklist, item count, and remind user that each run creates a new file. Summarize:
   - Focus areas selected
   - Depth level
   - Actor/timing
   - Any explicit user-specified must-have items incorporated

**Important**: Each `/speckit.checklist` command invocation creates a checklist file using short, descriptive names unless file already exists. This allows:

- Multiple checklists of different types (e.g., `ux.md`, `test.md`, `security.md`)
- Simple, memorable filenames that indicate checklist purpose
- Easy identification and navigation in the `checklists/` folder

To avoid clutter, use descriptive types and clean up obsolete checklists when done.

## Example Checklist Types & Sample Items

**UX Requirements Quality:** `ux.md`

Sample items (testing the requirements, NOT the implementation):

- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]"
- "Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]"
- "Are accessibility requirements specified for all interactive elements? [Coverage, Gap]"
- "Is fallback behavior defined when images fail to load? [Edge Case, Gap]"
- "Can 'prominent display' be objectively measured? [Measurability, Spec §FR-4]"

**API Requirements Quality:** `api.md`

Sample items:

- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"
- "Are authentication requirements consistent across all endpoints? [Consistency]"
- "Are retry/timeout requirements defined for external dependencies? [Coverage, Gap]"
- "Is versioning strategy documented in requirements? [Gap]"

**Performance Requirements Quality:** `performance.md`

Sample items:

- "Are performance requirements quantified with specific metrics? [Clarity]"
- "Are performance targets defined for all critical user journeys? [Coverage]"
- "Are performance requirements under different load conditions specified? [Completeness]"
- "Can performance requirements be objectively measured? [Measurability]"
- "Are degradation requirements defined for high-load scenarios? [Edge Case, Gap]"

**Security Requirements Quality:** `security.md`

Sample items:

- "Are authentication requirements specified for all protected resources? [Coverage]"
- "Are data protection requirements defined for sensitive information? [Completeness]"
- "Is the threat model documented and requirements aligned to it? [Traceability]"
- "Are security requirements consistent with compliance obligations? [Consistency]"
- "Are security failure/breach response requirements defined? [Gap, Exception Flow]"

## Anti-Examples: What NOT To Do

**❌ WRONG - These test implementation, not requirements:**

```markdown
- [ ] CHK001 - Verify landing page displays 3 episode cards [Spec §FR-001]
- [ ] CHK002 - Test hover states work correctly on desktop [Spec §FR-003]
- [ ] CHK003 - Confirm logo click navigates to home page [Spec §FR-010]
- [ ] CHK004 - Check that related episodes section shows 3-5 items [Spec §FR-005]
```

**✅ CORRECT - These test requirements quality:**

```markdown
- [ ] CHK001 - Are the number and layout of featured episodes explicitly specified? [Completeness, Spec §FR-001]
- [ ] CHK002 - Are hover state requirements consistently defined for all interactive elements? [Consistency, Spec §FR-003]
- [ ] CHK003 - Are navigation requirements clear for all clickable brand elements? [Clarity, Spec §FR-010]
- [ ] CHK004 - Is the selection criteria for related episodes documented? [Gap, Spec §FR-005]
- [ ] CHK005 - Are loading state requirements defined for asynchronous episode data? [Gap]
- [ ] CHK006 - Can "visual hierarchy" requirements be objectively measured? [Measurability, Spec §FR-001]
```

**Key Differences:**

- Wrong: Tests if the system works correctly
- Correct: Tests if the requirements are written correctly
- Wrong: Verification of behavior
- Correct: Validation of requirement quality
- Wrong: "Does it do X?"
- Correct: "Is X clearly specified?"
