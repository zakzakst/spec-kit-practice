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

## 概要 (Outline)

トリガーメッセージの `/speckit.specify` の後にユーザーが入力したテキストが、そのまま機能説明である。たとえ下に `$ARGUMENTS` が文字通り表示されていても、この会話内には説明がある前提で扱うこと。空コマンドでない限り、同じ説明を言い直させてはならない。

この機能説明を基に、次を行う。

1. **簡潔なショートネームの生成 (Generate a concise short name)**（2〜4 語）
   - 機能説明から意味の強いキーワードを抽出する
   - 本質を捉えた 2〜4 語のショートネーム (short name) を作る
   - 可能なら action-noun（動詞-名詞）形式にする
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

4. 次のフローで仕様を作成する。

   1. Input からユーザー説明を解析する
      - 空なら `ERROR "No feature description provided"`
   2. 主要概念を抽出する
      - アクター (actors)、アクション (actions)、データ (data)、制約 (constraints)
   3. 不明点への対応
      - 文脈と一般的慣行から妥当な仮定を置く
      - `[NEEDS CLARIFICATION: ...]` を付けてよいのは次の場合だけ
        - 機能スコープや UX に大きく影響する
        - 妥当な解釈が複数あり、含意が異なる
        - 妥当なデフォルトがない
      - **上限は 3 件**
      - 優先度は `スコープ (scope) > セキュリティ/プライバシー (security/privacy) > ユーザー体験 (user experience) > 技術的詳細 (technical details)`
   4. ユーザーシナリオ & テスト (User Scenarios & Testing) を埋める
      - 明確な user flow が導けない場合は `ERROR`
   5. 機能要件 (Functional Requirements) を作る
      - すべて検証可能であること
      - 未指定部分は合理的デフォルトを使い、前提条件 (Assumptions) へ記録する
   6. 成功基準 (Success Criteria) を定義する
      - 測定可能で技術非依存の成果にする
      - 定量 / 定性の両面を含める
      - 実装詳細抜きで検証可能であること
   7. データが関わるなら主要エンティティ (Key Entities) を特定する
   8. `SUCCESS (spec ready for planning)` を返す

5. テンプレート構造に従って仕様を `SPEC_FILE` へ書き込み、プレースホルダーを具体内容へ置き換える。セクション順と見出しは保つこと。

6. **仕様品質の検証 (Specification Quality Validation)**

   a. `FEATURE_DIR/checklists/requirements.md` に 仕様品質チェックリスト (Spec Quality Checklist) を作成する。構造はチェックリストテンプレートに従い、少なくとも次の検証項目を含める:

   ```markdown
   # 仕様品質チェックリスト (Specification Quality Checklist): [FEATURE NAME]

   **目的 (Purpose)**: 設計に進む前に仕様の完全性と品質を検証する
   **作成日 (Created)**: [DATE]
   **対象機能 (Feature)**: [Link to spec.md]

   ## コンテンツの品質 (Content Quality)

   - [ ] 実装の詳細（言語、フレームワーク、API）が含まれていないこと
   - [ ] ユーザー価値とビジネスニーズに焦点を当てていること
   - [ ] 非技術的な関係者向けに書かれていること
   - [ ] すべての必須セクションが入力されていること

   ## 要件の完全性 (Requirement Completeness)

   - [ ] [NEEDS CLARIFICATION] マーカーが残っていないこと
   - [ ] 要件がテスト可能で曖昧さがないこと
   - [ ] 成功基準が測定可能であること
   - [ ] 成功基準が技術に依存していないこと
   - [ ] すべての受け入れシナリオが定義されていること
   - [ ] エッジケースが特定されていること
   - [ ] スコープの境界が明確であること
   - [ ] 依存関係と前提条件が特定されていること

   ## 機能の準備状態 (Feature Readiness)

   - [ ] すべての機能要件に明確な受け入れ基準があること
   - [ ] ユーザーシナリオが主要なフローをカバーしていること
   - [ ] 成功基準で定義された測定可能な成果を機能が満たしていること
   - [ ] 仕様書に実装の詳細が漏れ出ていないこと

   ## メモ (Notes)

   - 未完了としてマークされた項目は、`/speckit.clarify` または `/speckit.plan` に進む前に仕様書の更新が必要です。
   ```

   b. チェックリストの各項目で spec をレビューし、pass / fail を判定する。問題があれば該当箇所を引用して記録する。

   c. **検証結果のハンドリング (Handle Validation Results)**
   - すべて pass ならチェック完了として次へ進む
   - `[NEEDS CLARIFICATION]` 以外で fail があるなら:
     1. 失敗した項目 (failing items) と具体的な問題を列挙
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

## 一般的なガイドライン (General Guidelines)

## クイックガイドライン (Quick Guidelines)

- ユーザーが**何を必要とし、なぜ必要なのか**に集中する
- 実装方法（技術スタック、API、コード構造）は書かない
- 読み手はビジネスステークホルダーであり、開発者ではない
- spec 内に埋め込みチェックリストを作らない

### セクション要件 (Section Requirements)

- **必須セクション (Mandatory sections)**: すべての機能で必須
- **オプションセクション (Optional sections)**: 関連する場合のみ含める
- 不要なセクションは `N/A` を残さず削除する

### AIによる生成について (For AI Generation)

1. 文脈、業界標準、共通パターンから合理的に補完する
2. 仮定は 前提条件 (Assumptions) セクションへ明記する
3. clarification（明確化質問）は最大 3 件までとし、本当に重要な論点に限る
4. 優先度は `スコープ (scope) > セキュリティ/プライバシー (security/privacy) > ユーザー体験 (user experience) > 技術的詳細 (technical details)`
5. テスターの視点で考え、曖昧な要件は不合格とみなす
6. 典型的に確認が必要なのは:
   - 機能スコープ / 境界 (feature scope / boundaries)
   - ユーザータイプ / 権限 (user types / permissions)
   - セキュリティ / コンプライアンス (security / compliance)

**合理的デフォルトの例**（通常は質問しない）

- データ保持期間 (Data retention): そのドメインの標準慣行
- パフォーマンス目標 (Performance targets): 一般的な Web / モバイルアプリ水準
- エラーハンドリング (Error handling): 分かりやすいメッセージと適切なフォールバック
- 認証方法 (Authentication method): 標準的な session-based または OAuth2
- 統合パターン (Integration patterns): 指定がなければ RESTful APIs

### 成功基準のガイドライン (Success Criteria Guidelines)

成功基準 (Success criteria) は次を満たすこと。

1. **測定可能であること (Measurable)**: 時間、割合、件数、レートなどの具体指標を含む
2. **技術に依存しないこと (Technology-agnostic)**: フレームワーク、言語、DB、ツール名を出さない
3. **ユーザー中心であること (User-focused)**: システム内部ではなくユーザー / ビジネス成果で書く
4. **検証可能であること (Verifiable)**: 実装詳細を知らなくても検証できる

**良い例 (Good examples)**

- `Users can complete checkout in under 3 minutes`（ユーザーは3分以内にチェックアウトを完了できる）
- `System supports 10,000 concurrent users`（システムは10,000人の同時接続ユーザーをサポートする）
- `95% of searches return results in under 1 second`（検索の95%が1秒以内に結果を返す）
- `Task completion rate improves by 40%`（タスク完了率が40%向上する）

**悪い例 (Bad examples)**

- `API response time is under 200ms`（APIのレスポンスタイムが200ms未満であること）
- `Database can handle 1000 TPS`（データベースが1000 TPSを処理できること）
- `React components render efficiently`（Reactコンポーネントが効率的にレンダリングされること）
- `Redis cache hit rate above 80%`（Redisのキャッシュヒット率が80%以上であること）
