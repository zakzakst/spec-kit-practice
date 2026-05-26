---
description: 自然言語の機能説明から、機能仕様を作成または更新する。
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
  - label: Clarify Spec Requirements
    agent: speckit.clarify
    prompt: Clarify specification requirements
    send: true
---

## User Input

```text
$ARGUMENTS
```

空でない場合、処理前にユーザー入力を**必ず**考慮すること。

## Outline

トリガーメッセージの `/speckit.specify` の後にユーザーが入力したテキストが、そのまま機能説明である。たとえ下に `$ARGUMENTS` が文字通り表示されていても、この会話内には説明がある前提で扱うこと。空コマンドでない限り、同じ説明を言い直させてはならない。

この機能説明を基に、次を行う。

1. **Generate a concise short name**（2〜4 語）
   - 機能説明から意味の強いキーワードを抽出する
   - 本質を捉えた 2〜4 語の short name を作る
   - 可能なら action-noun 形式にする
   - OAuth2、API、JWT などの技術語や略語は保持する
   - 短くても一目で分かる名前にする
   - 例:
     - `I want to add user authentication` -> `user-auth`
     - `Implement OAuth2 integration for the API` -> `oauth2-api-integration`
     - `Create a dashboard for analytics` -> `analytics-dashboard`
     - `Fix payment processing timeout bug` -> `fix-payment-timeout`

2. **既存 branch を確認してから新規作成する**

   a. まず最新の remote branch 情報を取得する

   ```bash
   git fetch --all --prune
   ```

   b. short-name に対して最大の feature number を 3 つの場所から探す
   - Remote branches: `git ls-remote --heads origin | grep -E 'refs/heads/[0-9]+-<short-name>$'`
   - Local branches: `git branch | grep -E '^[* ]*[0-9]+-<short-name>$'`
   - Specs directories: `specs/[0-9]+-<short-name>` に一致するディレクトリ

   c. 次に使う番号を決める
   - 3 ソースから番号を抽出
   - 最大値 N を取得
   - 新しい番号は `N+1`

   d. `.specify/scripts/powershell/create-new-feature.ps1 -Json "$ARGUMENTS"` を、算出した番号と short-name 付きで 1 回だけ実行する
   - `--number N+1`
   - `--short-name "your-short-name"`
   - feature description も一緒に渡す

   **IMPORTANT**
   - 3 ソースすべて確認すること
   - short-name が完全一致するものだけを数えること
   - 一致がなければ 1 から始めること
   - このスクリプトは feature につき 1 回しか実行してはならない
   - 実際の内容確認にはターミナルへ出る JSON を使うこと
   - JSON には `BRANCH_NAME` と `SPEC_FILE` が含まれる
   - シングルクォート入り引数は必要に応じてエスケープすること

3. `.specify/templates/spec-template.md` を読み、必須セクションを把握する。

4. 次のフローで仕様を作る。

   1. Input からユーザー説明を解析する
      - 空なら `ERROR "No feature description provided"`
   2. 主要概念を抽出する
      - actors, actions, data, constraints
   3. 不明点への対応
      - 文脈と一般的慣行から妥当な仮定を置く
      - `[NEEDS CLARIFICATION: ...]` を付けてよいのは次の場合だけ
        - 機能スコープや UX に大きく影響する
        - 妥当な解釈が複数あり、含意が異なる
        - 妥当なデフォルトがない
      - **上限は 3 件**
      - 優先度は `scope > security/privacy > user experience > technical details`
   4. User Scenarios & Testing を埋める
      - 明確な user flow が導けない場合は `ERROR`
   5. Functional Requirements を作る
      - すべて検証可能であること
      - 未指定部分は合理的デフォルトを使い、Assumptions へ記録する
   6. Success Criteria を定義する
      - 測定可能で技術非依存の成果にする
      - 定量 / 定性の両面を含める
      - 実装詳細抜きで検証可能であること
   7. データが関わるなら Key Entities を特定する
   8. `SUCCESS (spec ready for planning)` を返す

5. テンプレート構造に従って仕様を `SPEC_FILE` へ書き込み、プレースホルダーを具体内容へ置き換える。セクション順と見出しは保つこと。

6. **Specification Quality Validation**

   a. `FEATURE_DIR/checklists/requirements.md` に Spec Quality Checklist を作成する。構造はチェックリストテンプレートに従い、少なくとも次の検証項目を含める:

   ```markdown
   # Specification Quality Checklist: [FEATURE NAME]

   **Purpose**: Validate specification completeness and quality before proceeding to planning
   **Created**: [DATE]
   **Feature**: [Link to spec.md]

   ## Content Quality

   - [ ] No implementation details (languages, frameworks, APIs)
   - [ ] Focused on user value and business needs
   - [ ] Written for non-technical stakeholders
   - [ ] All mandatory sections completed

   ## Requirement Completeness

   - [ ] No [NEEDS CLARIFICATION] markers remain
   - [ ] Requirements are testable and unambiguous
   - [ ] Success criteria are measurable
   - [ ] Success criteria are technology-agnostic
   - [ ] All acceptance scenarios are defined
   - [ ] Edge cases are identified
   - [ ] Scope is clearly bounded
   - [ ] Dependencies and assumptions identified

   ## Feature Readiness

   - [ ] All functional requirements have clear acceptance criteria
   - [ ] User scenarios cover primary flows
   - [ ] Feature meets measurable outcomes defined in Success Criteria
   - [ ] No implementation details leak into specification

   ## Notes

   - Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
   ```

   b. チェックリストの各項目で spec をレビューし、pass / fail を判定する。問題があれば該当箇所を引用して記録する。

   c. **Handle Validation Results**
   - すべて pass ならチェック完了として次へ進む
   - `[NEEDS CLARIFICATION]` 以外で fail があるなら:
     1. failing items と具体問題を列挙
     2. spec を更新
     3. 最大 3 回まで再検証
     4. それでも fail が残る場合は checklist notes に残し、ユーザーへ警告
   - `[NEEDS CLARIFICATION]` が残る場合:
     1. marker を抽出
     2. 3 件超なら最重要 3 件だけ残し、残りは合理的仮定で埋める
     3. 各 clarification について、選択肢表付きの質問を作成する
     4. Markdown table は必ず正しく整形する
     5. `Q1`, `Q2`, `Q3` と連番にする
     6. すべての質問をまとめて提示する
     7. ユーザー回答を待つ
     8. 選択または自由記述に基づき marker を置換する
     9. clarification 解消後に再検証する

   d. 各検証イテレーション後、checklist ファイルを現在の pass / fail 状態で更新する

7. branch 名、spec パス、checklist 結果、次フェーズ (`/speckit.clarify` または `/speckit.plan`) へ進める状態かを報告する。

**NOTE:** スクリプトは branch 作成、checkout、spec ファイル初期化まで行う。書き込み内容はその後に上書きすること。

## General Guidelines

## Quick Guidelines

- ユーザーが**何を必要とし、なぜ必要なのか**に集中する
- 実装方法（技術スタック、API、コード構造）は書かない
- 読み手はビジネスステークホルダーであり、開発者ではない
- spec 内に埋め込みチェックリストを作らない

### Section Requirements

- **Mandatory sections**: すべての機能で必須
- **Optional sections**: 関連する場合のみ含める
- 不要なセクションは `N/A` を残さず削除する

### For AI Generation

1. 文脈、業界標準、共通パターンから合理的に補完する
2. 仮定は Assumptions セクションへ明記する
3. clarification は最大 3 件までとし、本当に重要な論点に限る
4. 優先度は `scope > security/privacy > user experience > technical details`
5. テスターの視点で考え、曖昧な要件は失格とみなす
6. 典型的に確認が必要なのは:
   - feature scope / boundaries
   - user types / permissions
   - security / compliance

**合理的デフォルトの例**（通常は聞かない）

- Data retention: そのドメインの標準慣行
- Performance targets: 一般的な Web / mobile app 水準
- Error handling: 分かりやすいメッセージと適切なフォールバック
- Authentication method: 標準的な session-based または OAuth2
- Integration patterns: 指定がなければ RESTful APIs

### Success Criteria Guidelines

Success criteria は次を満たすこと。

1. **Measurable**: 時間、割合、件数、レートなどの具体指標を含む
2. **Technology-agnostic**: フレームワーク、言語、DB、ツール名を出さない
3. **User-focused**: システム内部ではなくユーザー / ビジネス成果で書く
4. **Verifiable**: 実装詳細を知らなくても検証できる

**Good examples**

- `Users can complete checkout in under 3 minutes`
- `System supports 10,000 concurrent users`
- `95% of searches return results in under 1 second`
- `Task completion rate improves by 40%`

**Bad examples**

- `API response time is under 200ms`
- `Database can handle 1000 TPS`
- `React components render efficiently`
- `Redis cache hit rate above 80%`
